---
title: "Membuat File Watcher Dengan Systemd"
date: 2022-10-24T20:39:31+07:00
draft: true
tags:
- systemd
categories:
- Sysadmin
---

![Bismillah](/images/bismillah-2.png#center)

## Mukadimah

Pada artikel kali ini, saya akan membahas mengenai salah satu fitur Systemd yang memungkinkan user untuk menjalankan suatu _command_/_script_ atau bahkan mengeksekusi Systemd Service berdasarkan aktifitas suatu file/folder yang dimonitor.

Pernah dengan `incrond`? Nah, saya bisa katakan, systemd ini memiliki alternatifnya.

Jika merujuk pada halaman `man` terkait, fitur ini memanfaatkan API dari `inotify`. Sehingga semua limitasi yang ada pada `inotify`, juga berlaku berlaku pada komponen ini.

Komponen systemd yang dimaksud adalah `systemd.path`.


