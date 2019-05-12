# FP_SISOP19_B06

## Oleh Kelompok B06 :
1. Yasinta Yusniawati   05111740000054
2. Bima Satria Ramadhan 05111740000081

### SOAL
Buatlah program C yang menyerupai crontab menggunakan daemon dan thread. Ada sebuah file crontab.data untuk menyimpan config dari crontab. Setiap ada perubahan file tersebut maka secara otomatis program menjalankan config yang sesuai dengan perubahan tersebut tanpa perlu diberhentikan. Config hanya sebatas * dan 0-9 (tidak perlu /,- dan yang lainnya)

#### Jawaban
Dalam program yang kami buat terdapat 3 fungsi :
1. Main (Membuat daemon dan membaca crontab.data)
2. Cek (Mengecek apakah waktu yang ada di crontab.data sama dengan waktu sekarang dan memisah antara format waktu dengan command)
3. Run (karena kami hanya copast di modul nama fungsi di programnya adalah **print_message_function**)

##### Penjelasan Program
1. Daemon

```c
pid_t pid, sid;

  pid = fork();

  if (pid < 0) {
    exit(EXIT_FAILURE);
  }

  if (pid > 0) {
    exit(EXIT_SUCCESS);
  }

  umask(0);

  sid = setsid();

  if (sid < 0) {
    exit(EXIT_FAILURE);
  }

  if ((chdir("/")) < 0) {
    exit(EXIT_FAILURE);
  }

  close(STDIN_FILENO);
  close(STDOUT_FILENO);
  close(STDERR_FILENO);

  pthread_t thread[100];
```

2. Mengecek waktu sekarang

```c
    time_t t=time(NULL);
	  struct tm tm = *localtime(&t);
    
    int menit = tm.tm_min;
    int jam = tm.tm_hour;
    int tanggal = tm.tm_mday;
    int bulan = tm.tm_mon + 1; 
    int hari = tm.tm_wday;
```
Waktu diambil menggunakan fungsi localtime yang disimpan pada struct tm. Setiap menit, jam, tanggal, bulan dan hari masing-masing disimpan pada variable tersendiri.

3. Membuka file crontab.data

```c
FILE *fp;
  
    fp = fopen("/home/yasinta/crontab.data", "r"); // read mode
  
    if (fp == NULL)
    {
        perror("Error while opening the file.\n");
        exit(EXIT_FAILURE);
    }
  
```
Digunakan untuk membuka file crontab.data dalam hal ini file crontab.data tersimpan di /home/yasinta/crontab.data. Dan jika file crontab.data belum ada akan mengeluarkan pesan error.

4. Membuat thread dan memangsil fungsi check

```c
char isi[1000];
    int i=0;
    int join;
    while (fgets(isi, 1000, fp) != NULL)  {
      char command_dijalankan[1000];
      if(check(tm,isi,command_dijalankan)){
        // printf("%s\n",command_dijalankan);
        int iret = pthread_create( &thread[i], NULL, print_message_function, (void*) command_dijalankan);
        i++;
      }
    }
    join = i;
    for(int p=0; p<join; p++)
    {
      pthread_join( thread[p], NULL);
    }
    fclose(fp);
```
Program di atas akan mengambil isi dari file crontab.data dan mengeceknya dalam 1 baris. Jika isi dari file crontab.data sama dengan waktu sekarang maka command yang ada dalam file crontab.data akan di jalankan. Selanjutknya semua thread akan dibuat, setelah semua thread terbuat thread tersebut akan di join.

5. Memset

```c
char space[6][4];
  memset(space[0],'\0',sizeof(space[0]));
  memset(space[1],'\0',sizeof(space[0]));
  memset(space[2],'\0',sizeof(space[0]));
  memset(space[3],'\0',sizeof(space[0]));
  memset(space[4],'\0',sizeof(space[0]));
```
Berguna untuk mengosongkan array yang nanti akan di isi oleh file crontab.data

6. Memisah space berdasarkan format crontab
```c
int counter = 0;
  int digit = 0;
  int iter = 0 ;
  while(buffer[iter] != '\0' && counter < 5)
  {
    if(buffer[iter]==' '){
      counter++;
      digit = 0;
    }
    else {
      space[counter][digit] = buffer[iter];
      digit++;
    }
    iter++;
  }
  strcpy(cmd,buffer+iter);
```
Memisahkan format waktu dan command. Misal ``* * * * * cp /home/yasinta/music /home/yasinta/baru`` menjadi ``* * * * *`` yang disimpan pada variable space dan ``cp /home/yasinta/music /home/yasinta/baru`` yang disimpan pada variable command_dijalankan yang terdapat pada fungsi main program.

7. Cek waktu sekarang dengan isi crontab.data

```c
 char menit[3]; char jam[3]; char tanggal[3]; char bulan[3]; char hari[3];

  sprintf(menit, "%d", tm.tm_min);
  sprintf(jam, "%d", tm.tm_hour);
  sprintf(tanggal, "%d", tm.tm_mday);
  sprintf(bulan, "%d", tm.tm_mon+1);
  sprintf(hari, "%d", tm.tm_wday);

  int flag = 1;

  if(strcmp(menit, space[0]) != 0 && strcmp(space[0], "*") != 0) return 0;
  if(strcmp(jam, space[1]) != 0 && strcmp(space[1], "*") != 0) return 0;
  if(strcmp(tanggal, space[2]) != 0 && strcmp(space[2], "*") != 0) return 0;
  if(strcmp(bulan, space[3]) != 0 && strcmp(space[3], "*") != 0) return 0;
  if(strcmp(hari, space[4]) != 0 && strcmp(space[4], "*") != 0) return 0;

  return flag;
 ```
 setiap waktu yang ada pada struct tm akan di simpan pada variable menit, jam, tanggal, bulan dan hari yang berupa string. Dimana jika waktu nya di file crontab.data berbeda dengan waktu sekarang dan bukan merupakan * makan return false yang artinya bukan waktunya. Namun akan return true jika merupakan waktu eksekusi crontab.
 
 8. Menjalankan Command
 
 ```c
 void *print_message_function( void *ptr )
{
    char *message;
    message = (char *) ptr;
    // printf("%s\n",message);
    system(message);
    pthread_exit(NULL);
}
 ```
 Fungsi ini dipanggil pada thread yang dibuat. Yang berfungsi untuk menjalankan command.
