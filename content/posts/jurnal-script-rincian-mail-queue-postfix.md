---
title: "Jurnal: Script Rincian Mail Queue Postfix"
date: 2022-04-08T09:25:02+07:00
tags:
- scripting
- bash
- python
categories:
- Jurnal
---

![Bismillah](https://drive.google.com/uc?export=download&id=17WTklzV4j4O0PMOeHtAqjXjXd9hrtfbT#center)

Pada artikel ini saya akan memberikan contoh script Bash dan Python yang berfungsi untuk menampilkan jumlah antrian email (_mail queue_) keluar pada Postfix. Ini berguna sebagai metrics yang bisa digunakan untuk alerting. Baik alerting manual maupun terintegrasi seperti ServerDensity, dan sebagainya.

### Dasar Script

Komponen utama utama untuk mengetahui _mail queue_ adalah folder dimana _queue_ tersebut singgah untuk sementara waktu. Untuk mengetahuinya, kita bisa memanggil command `postconf -h queue_directory` yang outputnya akan menunjukkan dimana direktori tersebut.

Contoh struktur dari folder queue tersebut adalah seperti ini:

```
root@postfix-relay-new:/var/spool/postfix# tree -d -L 1 -n
.
├── active
├── bounce
├── corrupt
├── defer
├── deferred
├── dev
├── etc
├── flush
├── hold
├── incoming
├── lib
├── maildrop
├── pid
├── private
├── public
├── saved
├── trace
└── usr

18 directories
```

Folder-folder yang berisi _queue_ email adalah active, corrupt, defer, deferred, hold dan incoming.

Contoh file _queue_ yang ada pada folder deferred:
```
root@postfix-relay-new:/var/spool/postfix/deferred# find -type f
./E/EAB4426846
./7/7C31723297
./B/B2F962B543
...
./2/28BD526F02
./8/8B6742C09D
./5/548D72A868
```

Sehingga dari situ, kita cukup meracik _command_ `find` sedemikian rupa sehingga _command_ tersebut hanya akan menampilkan file-file queue saja pada direktori yang sudah ditunjuk oleh `postconf -h queue_directory`. Karena ciri file _queue_ sifatnya seragam, maka bisa kita gunakan Regular Expression dibawah ini:

```regex
[0-9A-F]+
```

Kemudian, jika sudah dapat outputnya, maka untuk masing-masing kondisi _queue_ (active/defer, dsb) tinggal dihitung ada berapa baris yang pada _path_ -nya terdapat substring *active* misalnya. Atau *defer*, dll. Dan ini bisa dilakukan dengan berbagai macam cara. Salah satu diantaranya adalah dengan `grep`.

```bash
grep -w -c 'active' input

# Dimana:
# -w  mencari kata utuh, bukan substring
# -c  jumlah temuan
```

Dan selesai, intinya hanya itu saja.

### Versi Bash

```bash
#!/usr/bin/env bash

readonly queue_dir=$(postconf -h queue_directory) 
readonly tmp_file=$(mktemp) 

states=(all active corrupt defer deferred hold incoming) 

get_count() {
    printf '%s: %d\n' "$1" $(egrep -w -c "$2" "$tmp_file") 
} 

main() {
    find "$queue_dir" -type f -regextype egrep -regex '.*/[0-9A-F]{5,}$' > "$tmp_file"
    
    for state in ${states[@]}; do
        search="$state" 
        [[ $state = all ]] && search='.*'
        get_count "$state" "$search"
    done

    rm -f "$tmp_file"
}

main 
```

Contoh output jika dijalankan:
```
root@postfix-relay-new:~/script# bash mqueue.sh
all: 41
active: 0
corrupt: 0
defer: 27
deferred: 14
hold: 0
incoming: 0
```

### Versi Python

```python
import os
from subprocess import check_output

def get_list():
    queue_dir = check_output(['postconf','-h','queue_directory']).decode('utf-8').rstrip()
    command_string = 'find ' + queue_dir + ' -type f -regextype egrep -regex .*/[0-9A-F]+$'
    return check_output(command_string.split()).decode('utf-8').rstrip().split('\n')

def get_count(string, search, list):
    n = sum(search in entry for entry in list)
    print(f'{string}: {n}')

if __name__ == '__main__':
    result = get_list()
    states = ['all', 'active', 'corrupt', 'defer', 'deferred', 'hold', 'incoming']
    
    for state in states:
        search = state + '/' if state != 'all' else ''
        get_count(state, search, result)
```

Contoh output jika dijalankan:
```
root@postfix-relay-new:~/script# python3 mqueue.py 
all: 41
active: 0
corrupt: 0
defer: 27
deferred: 14
hold: 0
incoming: 0
```

---

Semoga bermanfaat, _insyaAllah_.
