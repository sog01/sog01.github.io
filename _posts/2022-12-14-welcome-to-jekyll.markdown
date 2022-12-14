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

Buka URL berikut http://localhost:9200 di dalam browser Kamu. Jika, Kamu mendapatkan tampilan dengan *tagline* "You Know, for Search" selamat, Kamu telah berhasil menjalankan `Elasticsearch` di komputer lokalmu!

{% include image.html url="/assets/images/get-localhost.png" description="You Know, for Search" %} 

Lalu, Kamu dapat mematikan nya dengan menjalankan perintah dari `docker` seperti di bawah ini :

{% highlight bash %}
docker stop es01
{% endhighlight %}


## Men-Setup Client
Kita harus mensiapkan sebuah `Go` *HTTP* klien untuk dapat berkomunikasi dengan `Elasticseach`. Yuk, Kita inisialisasi objek klien dengan fungsi *health check* sebagai berikut :

{% gist ac59a1ffc57b0109125a7ced699d09c3 %}

## Membuat sebuah Indeks
Setelah *Go* klien sudah siap, Kita dapat melanjutkan untuk membuat indexs pertama Kita. Indexs itu seperti *schema* di `SQL Database`. Tapi, sebelum melakukan itu, Kita harus men-desain data model Kita terlebih dahulu. Mari, Kita gunakan pegawai sebagai data model Kita :

{% gist 92d19df23065773c08005bc841b43c6b %}

Sekarang, Kita sudah siap untuk membuat indexs baru Kita. Mari, Kita membuat indeks dengan cara meng-*Hit* endpoint `PUT /employee` dari Elasticsearch API :

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

Selanjutnya, Kita dapat memeriksa pegawai indeks bahwa berhasil Kita buat atau belum dengan cara meng-hit `GET /_cat/indices`.

{% include image.html url="/assets/images/check-index-exists.png" description="Menjukkan Indeks yang ada" %} 

Akhirnya, Kita dapat menambahkan fungsi *CreateIndex* di dalam klien *code* Kita :

{% gist cefe85b72c0e7ebd1a81c6bd110cb061 %}

## Insert beberapa Data

Setelah, Kita membuat pegawai index, Kita dapat memulai untuk memasukkan beberapa data. 

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

Untuk memastikan data berhasil dimasukkan
