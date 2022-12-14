---
layout: post
title:  "Elasticsearch Getting Started with Go"
date:   2022-12-14 06:09:25 +0700
categories: elasticsearch
---

{% include image.html url="/assets/images/gopher-x-es.png" %} 


Halo teman - teman, ini adalah artikel pertamaku tentang bagaimana memulai menggunakan `Elasticsearch` dan `Go`. Ini meliputi bagaimana membangun *HTTP server* sederhana yang dapat melakukan operasi *CRUD (Create, Read, Update, and Delete)*. Setelah membaca artikel ini, Saya harap Kamu dapat membuat *project* yang keren menggunakan *ElasticSearch x Go*. Jadi, tanpa menunggu lama lagi, langsung aja gaskeun!

## Apa itu Elasticsearch
`Elasticsearch` adalah layanan *open-source* yang dapat digunakan untuk membangun sebuah mesin pencarian. Layanan ini *scalable* dan *robust* untuk menangani jumlah rikues pencarian dalam jumlah besar. Banyak perusahaan teknologi besar (seperti Netflix, Microsoft, dan lain - lain) yang telah mengimplementasikan `Elasticsearch` untuk kebutuhan mereka. Karena itu, jika Kamu ingin membangun sebuah mesin pencarian yang *powerfull*, Kamu harus mencoba yang satu ini.

## Menjalankan Elasticsearch dengan Docker
Terdapat banyak cara untuk menjalankan `Elasticsearch` *server*. Menggunkan `docker` adalah cara yang paling mudah. Dengan ini Kamu dapat dengan mudah meng-*uninstall* nya dengan cara menghapus *container* di dalam `docker`. Jika Kamu belum meng-*install* docker, jangan khawatir; Kamu dapat mengikut langkah - langkah dari dokumentasi `docker` sebagai berikut :
- [Menginstall docker dari Linux](https://docs.docker.com/engine/install/ubuntu/)
- [Menginstall docker dari Mac](https://docs.docker.com/desktop/install/mac-install/)
- [Menginstall docker dari Windows](https://docs.docker.com/desktop/install/windows-install/)

Setelah process installasi nya selesai, Kamu dapat menjalankan perintah berikut untuk menghidupkan *server* dari `Elasticsearch` :
{% highlight bash %}
docker run -d --name es01 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -it docker.elastic.co/elasticsearch/elasticsearch:7.14.0
{% endhighlight %}

Buka URL berikut `http://localhost:9200` di dalam browser Kamu. Jika, Kamu mendapatkan tampilan dengan *tagline* "You Know, for Search" selamat, Kamu telah berhasil menjalankan `Elasticsearch` di komputer lokalmu!

{% include image.html url="/assets/images/get-localhost.png" description="You Know, for Search" %} 

Lalu, Kamu dapat mematikan nya dengan menjalankan perintah dari `docker` seperti di bawah ini :

{% highlight bash %}
docker stop es01
{% endhighlight %}


## Men-Setup Client
Kita harus mensiapkan sebuah `Go` *HTTP* klien untuk dapat berkomunikasi dengan `Elasticseach`. Yuk, Kita inisialisasi objek klien dengan fungsi *health check* sebagai berikut :

{% gist ac59a1ffc57b0109125a7ced699d09c3 %}

## Membuat sebuah Indeks
Setelah *Go* klien sudah siap, Kita dapat melanjutkan untuk membuat indeks pertama Kita. Indeks itu seperti *schema* di `SQL Database`. Tapi, sebelum melakukan itu, Kita harus men-desain data model Kita terlebih dahulu. Mari, Kita gunakan pegawai sebagai data model Kita :

{% gist 92d19df23065773c08005bc841b43c6b %}

Sekarang, Kita sudah siap untuk membuat indeks baru Kita. Mari, Kita membuat indeks dengan cara memanggil endpoint `PUT /employee` dari Elasticsearch API :

{% highlight bash %}
curl --location --request PUT 'http://localhost:9200/employee' \
--header 'Content-Type: application/json' \
--data-raw '{
    "mappings": {
        "properties": {
            "id": {
                "type": "integer"
            },
            "name": {
                "type": "text"
            },
            "address": {
                "type": "text"
            },
            "salary": {
                "type": "float"
            }
        }
    }
}'
{% endhighlight %}

Selanjutnya, Kita dapat memeriksa indeks dengan cara meng-hit `GET /_cat/indices`.

{% include image.html url="/assets/images/check-index-exists.png" description="Menjukkan Indeks yang ada" %} 

Akhirnya, Kita dapat menambahkan fungsi *CreateIndex* di dalam klien *code* Kita :

{% gist cefe85b72c0e7ebd1a81c6bd110cb061 %}

## Insert beberapa Data

Setelah, Kita membuat pegawai indeks, Kita dapat memulai untuk memasukkan beberapa data. 

Kita dapat menggunakan *API* `PUT employee/_doc/1` dengan *payload* sebagai berikut :

{% highlight bash %}
curl --location --request PUT 'http://localhost:9200/employee/_doc/1' \
--header 'Content-Type: application/json' \
--data-raw '{
"id": 1,
"name": "Abdullah",
"address": "Madinah, Saudi Arabia",
"salary": 1000000
}'
{% endhighlight %}

Untuk memastikan data berhasil dimasukkan, Kita dapat memanggil `GET employee/_doc/1` untuk mengambil data dari *Elasticsearch*.

{% include image.html url="/assets/images/get-employee-data.png" description="Mengambil data Pegawai yang Pertama" %} 

Untuk terlihat *realistic*, Kita harus memasukkan beberapa data pegawai dengan cara menggunakan perulangan untuk memanggil `PUT employee/_doc/{id}` *API* dengan random *id*. Kita dapat meringkas nya dengan cara membuat fungsi `InsertData` dan `SeedingData` :

{% gist 29a24e22707ac3e03f3406cfb79ca702 %}

## Mencari Data
Operasi pencarian merupakan andalan dari `Elasticsearch`. Kita dapat melakukan kompleks pencarian seperti mesin pencarian pada umumnya. Tetapi untuk saat ini, Kita hanya melakukan process pencarian sederhana mengunakan operasi [match](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html) :

{% highlight bash %}
curl --location --request GET 'http://localhost:9200/employee/_search' \
--header 'Content-Type: application/json' \
--data-raw '{
    "query": {
        "match": {
            "name": "abdullah"
        }
    }
}'
{% endhighlight %}

Pencarian diatas mencari pegawai berdasarkan kata kunci nama yang di berikan. Process ini membagi kata kunci menjadi beberapa kata dan kemudian mencocokannya dengan data yang ada di dalam `Elasticsearch`.

Hasilnya, akan memberikan daftar dari pegawai jika nama mereka cocok dengan kata kunci yang diberikan.

{% include image.html url="/assets/images/search-result.png" description="Mencari data menggunakan postman" %} 

Kemudian, mari Kita tambahkan operasi *match* ini kedalan kode klien Kita di dalam fungsi *SearchData* :

{% gist 42480dd9e163d621f940d26db00001bb %}

Selebihnya, operasi pencarian hanya dapat di lakukan meggunakan tipe [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) karena tipe ini adalah tipe yang spesial yang dapat melakukan teknik *full-text* proses.

## Mengganti Data

Kita dapat memanggil *endpoint* `POST employee/_update/{id}` untuk mengganti data pegawai Kita. Kita hanya mengganti sebagian data dari *endpoint* ini. Karena itu, Kita dapat menentukan data yang hanya akan Kita ganti. Di *code* berikut, Kita hanya mengganti nama pegawai Kita : 


{% highlight bash %}
curl --location --request POST 'http://localhost:9200/employee/_update/1' \
--header 'Content-Type: application/json' \
--data-raw '{
"doc": {
   "name": "Abdullah Ja'far"
 }
}'
{% endhighlight %}

Untuk memastikan data berhasil di ganti, Kita dapat memanggil `GET employee/_doc/{id}` untuk memeriksanya.

{% include image.html url="/assets/images/get-data-after-update.png" %} 

Lalu, Kita bisa tuliskan `code` diatas menggunakan Go di dalam fungsi `UpdateData` :

{% gist b2c4113e613008bd77e56e9a8d5e79ab %}

## Menghapus Data
Kita dapat memanggil `DELETE employee/_doc/{id}` *endpoint* jika Kita ingin menghapus data pegawai :

{% highlight bash %}
curl --location --request DELETE 'http://localhost:9200/employee/_doc/1'
{% endhighlight %}

Lalu, Kita tuliskan di dalam kode klien Go di dalam fungsi `DeleteData` :

{% gist e11d00d835decf2e71fd4c3023f9e8b2 %}

## CRUD Server

Akhirnya, Kita bisa menggabungkan semua operasi *CRUD* kedalam *code HTTP Server* Kita :

{% gist 2feec2669764e13d96cb458afd540ac5 %}

Server ini membuka *port* nya di 8080. Kita dapat mencobanya dengan memanggil `http://localhost:8080/health` di browser Kita untuk memastika *server* nya bekerja.

{% include image.html url="/assets/images/health-check-ok.png" %}

Mari Kita coba *server* Kita dengan melakukan beberapa aksi sebagai berikut:

### Memasukkan Data Pegawai
{% include image.html url="/assets/images/insert-data-from-crud-server.png" %}

### Mengganti Data Pegawai
{% include image.html url="/assets/images/update-data-from-crud-server.png" %}

### Mencari Data Pegawai
{% include image.html url="/assets/images/search-data-from-crud-server.png" %}

### Menghapus Data Pegawai
{% include image.html url="/assets/images/delete-data-from-crud-server.png" %}

## Kesimpulan
Selamat, *CRUD Server* Kita berhasil berjalan! Kamu dapat mencobanya untuk mendapatkan beberapa ide mengenai bagaimana mengimplementasikan `Elasticsearch` di proyek kerenmu. Kamu juga dapat mendapatkan kode lengkap nya di [Github repository](https://github.com/sog01/getting-started-elastic-search-go) Saya.

Terakhir, terimakasih telah membaca artikel ini, Saya harap ini membantu Kamu untuk memahami dasar tentang `Elasticsearch`. Dan, tetap pantau Saya, Karena Saya akan membagikan topik yang lebih *advanced* kedepannya.

