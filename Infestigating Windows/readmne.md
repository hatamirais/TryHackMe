Investigating Windows

https://tryhackme.com/room/investigatingwindows

Konek ke komputer target melalui RDP dengan:

IP:			IP Address di Active Machine Information
!!cantumin gambar!!
Username:	Administrator
Password:	letmein123

1 Whats the version and year of the windows machine?

klik logo windows > settings > about


2 Which user logged in last?

Klik logo windows dan ketik `Event Viewer`
Pilih `Windows Logs` - `Security`. Di bagian `Number of events` pilih yang paling bawah dimana tanggalnya adalah `2/13/2019`.

Di bagian `Event 1102,Eventlog` pilih `detail` dan pilih XML View, `ctrl+f` dan cari `user`


3 When did John log onto the system last?

Answer format: MM/DD/YYYY H:MM:SS AM/PM

Buka `Command Prompt` atau `cmd` dan ketikkan `net user  #username_yang_dicari`



4 What IP does the system connect to when it first starts?

Tunggu program `C:\TMP\p.exe` muncul, ada di `connecting to`, jika tidak ada klik `terminate` dan `start machine` lagi, tunggu hingga bootup dan `C:\TMP\p.exe` akan muncul

Jawabannya 10.34.2.3



5 What two accounts had administrative privileges (other than the Administrator user)?

Answer format: username1, username2

`net localgroup Administrators`



6 Whats the name of the scheduled task that is malicous.

Ketik `Task Scheduler`




7 What file was the task trying to run daily?

Masih menggunakan program yang sama dengan yang di atas, lihat di tab `Actions` di task yang malicous



8 What port did this file listen locally for?

Masih menggunakan program yang sama dengan yang di atas, lihat di tab `Actions` di task yang malicous




9 When did Jenny last logon?

Buka `Command Prompt` atau `cmd` dan ketikkan `net user  #username_yang_dicari`



10 At what date did the compromise take place?

Answer format: MM/DD/YY

Masih menggunakan data yang sama dengan yang di atas


11 At what time did Windows first assign special privileges to a new logon?

Answer format: MM/DD/YYYY HH:MM:SS AM/PM

Gunakan proggram `Event Viewer` seperti pada pertanyaan kedua
Di pertanyaan, di tanyakan tentang kapan Windows pertama kali meng-assign special privileges ke new logon, jika anda googling dengan keyword `Windows assign special privileges to logon` anda akan menemukan kode log event `4672`. Sort `Event ID` hanya untuk `4672`, anda akan menemukan banyak log di tanggal 03/02/2019. Buka `HINT` di website dan jawabannya ada di situ


12 What tool was used to get Windows passwords?

Tools yang umumnya di gunakan untuk membobol password sudah di deteksi oleh Windows Defender sebagai aplikasi yang berbahaya. Karena itu penting untuk memeriksa `Event Viewer`

Buka `Event Viewer` - `Application and Security` - `Microsoft` - `Windows` -`Windows Defender` - `Operational`

Sort dari waktu awal, anda akan menemukan informasi `Warning` pada tab `Level`, di situ anda akan menemukan beberapa program yang di tandai sebagai `Trojan` dan `Tool` yang berbahaya, salah satunya `Mikatz!dha`. Jika anda masukkan `Mikatz!dha` sebagai jawaban, salah. `Mikatz!dha` adalah nama lain program ini jika berjalan di Windows Platform. Googling `Mikartz!dha` dan anda akan dapat jawabannya



13 What was the attackers external control and command servers IP?

IP apa yang di gunakan oleh hacker untuk komunikasi dengan komputernya. Setiap server memiliki IP unik agar bisa di temukan oleh komputer lain, karena ada sekitar 4 miliar IP yang bisa di gunakan sebagai server tidak mungkin untuk di periksa semuanya.

Di windows ada file yang bernama `host` yang letaknya umumnya di `C:\Windows\System32\drivers\etc` yang bisa di gunakan salah satunya untuk terhubung dengan 1 server tertentu. Contohnya dulu untuk mengakses Reddit ada 2 cara, VPN dan edit file `host` ini dengan menambahkan alamat IP Reddit di file `host` ini dan bisa melewati blockirnya Indonesia. **TLDR** periksa file `host` karena mungkin ada informasi yang menarik di sana

`C:\Windows\System32\drivers\etc` - buka `host` dengan notepad. Jawabannya ada di dalamnya.


14 What was the extension name of the shell uploaded via the servers website?

Periksa `C:\\inetpub\wwwroot`


15 What was the last port the attacker opened?

`Start Menu` - `Firewall` - `Advanced Settings` - `Inbound Rules`

Ada 1 port untuk protokol TCP yang jarang di gunakan, 4 digit

16 Check for DNS poisoning, what site was targeted?
Menggunakan jawaban dari nomor 13, karena sama-sama ada kaitannya dengan DNS poisoning


src: https://jiosurguladze.medium.com/investigating-windows-thm-5ca486b50a17