# **İÇİNDEKİLER**
<html>
<head>
</head>
<body>

<li><a href="#ATS">ATS: Veritabanı Yönetim Sistemi</a></li>
<ul type="circle"><li><a href="#Veritabanı_olusturma">Veritabanı Oluşturma</a></br>
    <li><a href="#Veritabanı_silme">Veritabanı Silme</a></li></ul><br>



<li><a href="#Postgresql">PostgreSQL</a></li> </br> 




<ul type="circle"><li><a href="#HyperV">HyperV</a></br>
    <li><a href="#HyperV_üzerinde_kurulumu">HyperV üzerinde kurulumu</a></li></ul>
<ul type="circle"><li><a href="#Kubernetes">Kubernetes</a></br>
    <li><a href="#Kubernetes_üzerinde_kurulumu">Kubernetes üzerinde kurulumu</a></li>
    <li><a href="#Kubernetes_uzerinde_baglanti_bilgilerinin_edinilmesi">Kubernetes üzerinde bağlantı bilgilerinin edinilmesi</a></ul></li><br>


<li><a href="#Faydali_linkler">Faydalı Linkler</a></li>
<li><a href="#Tablolar">Tablolar</a></li>
---------------------------------------------------------------------------
<h2 id="ATS"></h2>

# **ATS: Veritabanı Yönetim Sistemi**
<li>ATS projesinde veritabanı olarak PostgreSQL kullanılmıştır.</li>
<li>Veritabanı linux üzerinde servis olarak, docker içinde veya kubernetes üzerinde çalıştırılabilir. ATS projesinde veritabanı Kubernetes içinde çalışmaktadır.</li>
<li>Veritabanı yedekliliği Patroni yazılımı ile sağlanmaktadır.</li>
<li>Kubernetes kurulumu için zalando postgres operatör kullanılmaktadır.</li>
<li>Veritabanı 3 kopya olacak şekilde yedekli çalışmakta ve senkron mode kullanılmaktadır.</br></li>


<h2 id="Veritabanı_olusturma"></h2>

## **Veritabanı Oluşturma**
<li> Türkçe karater seti ile uyumlu kullanabilmek için veri tabanını oluştururken _"LC_COLLATE='tr_TR.UTF-8'" ve "LC_CTYPE='tr_TR.UTF-8'"_ özelliğini değiştirmemiz gerekir.</li>


```
CREATE DATABASE "ATS_Test" WITH
OWNER = postgres
ENCODING = 'UTF8'
LC_COLLATE='tr_TR.UTF-8'
LC_CTYPE='tr_TR.UTF-8'
TABLESPACE = test
TEMPLATE=template0;
```

<li>Test etmek için aşağıdaki kodu kullanabiliriz:</li>

```
select upper('Aaıiğüşöç'),lower('AaİĞÜŞİÖÇ')
```

<h2 id="Veritabanı_silme"></h2>

## **Veritabanı Silme**
<li>Postgresql üzerinden direk veritabanını silemezsiniz. İlk önce veritabanına bağlantıyı kapatmanız gerekir. Daha sonra bağlı bulunan bağlantıları kapatmanız gerekir. Daha sonra drop komutu ile veri tabanını silebilirsiniz.</li>

 ```
UPDATE pg_database SET datallowconn = 'false' WHERE datname = 'ATS_Test';
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'ATS_Test';
DROP DATABASE "ATS_Test";
```


<h2 id="Postgresql"></h2>


# **POSTGRESQL**
<h2 id="HyperV"></h2>

## **HyperV** <br/>
<h2 id="HyperV_üzerinde_kurulumu"></h2>

### **HyperV üzerinde kurulumu** <br/>
 
<li>Ubuntu sunucusunda kurulum yapmak için;</p></li>

<ul type="circle"><li>Sunucuyu güncellememiz gerekir.</li></ul>








```
sudo apt-get update
```




<li>PostgreSQL veritabanını yüklemek için aşağıdaki komut çalıştırılır.</li>


 
```
sudo apt-get install postgresl postgresql-contrib
```





<li>Veritabanının yüklendiğini kontrol etmek için aşağıdaki yolu kontrol edebilirsiniz. Kurulan sürüme göre yol değişmektedir.</li>

```
ls /etc/postgresql/{Version}/main/
```




<li>Servisin durumunu öğrenmek ve yeniden başlatma vb. işlemleri yapmak için aşağıdaki komutları kullanabiliriz.</li>

```
service postgresql status => Servis durumu 
service postgresql restart => Servisi yeniden başlatma
service postgre start => Servisi başlatma
service postgresql stop => Servisi durdurma 
```




<li>postgres kullanıcısına şifre oluşturmak için postgres kullanıcısına geçmeliyiz.</li>

```
sudo su postgres
```


<li>postgres kullanıcısına geçtikten sonra veritabanına bağlamamız gerekir. Onun için aşağıdaki komutu çalıştırırız.</li>

```
psql
```



<li>Veritabanına bağlandıktan sonra şifre değiştirmek içi aşağıdaki sql komutunu çalıştırırız.</li>

```
ALTER USER postgres WITH PASSWORD 'test123';
CREATE USER user_1 WITH PASSWORD 'test123';
```
<ul type="circle"><li>\du komutu ile oluşturulan tablo görüntülenir.</li></ul>

```
CREATE USER user_2 WITH PASSWORD 'test123';
```


<li>Dosyaya erişim sağlamadan önce, dosyaya yazı yazma erişimi kapalı olabilir. Aşağıdaki komut ile istenen dosyaya hem okuma hem de yazma yetkisi verilir.</li></br>


```
sudo chmod 777 /etc/postgresql/14/main/postresql.conf -R
```


<li>Veritabanının şifreli giriş şekline dönüştürmek için config dosyalarında bulunan <span style="color:red;"> pg_hba.conf </span> dosyasının içerisine aşağıdaki satırı eklemeliyiz.</li>

```
host all all ::1/128 md5
host all all 0.0.0.0/0 trust #md5
```
<ul type="circle"><li>Satır ekledikten sonra postgresql servisini yeniden başlatmamız gerekir.</li></ul>



<li>Veritabanını dış sistemlere açılması için config klasöründe bulunan <span style="color:red;">postgresql.conf </span> dosyasının içerisinde yer alan <u>listen addresses</u> alanını aşağıdaki gibi değiştirmemiz gerekir.</li>

<ul type="circle"><li><u>listen addresses='*'</u></li></ul>
<ul type="circle"><li>Değişikliği yaptıktan sonra postgresql servisini yeniden başlatmamız gerekir.</li></ul>

<li>Veri tabanı servisi kurulduktan sonra veriyi restore etmeden önce <span style="color:red;">tr_TR.UTF-8 </span> dil paketinin Ubuntu sistemine kurulması gerekmektedir.</li>

<ul type="circle"><li>Mevcut paketleri listeleme </li></ul>

```
locale -a
```
<ul type="circle"><li>Root kullanıcısına geçilir.</li></ul>

```
sudo -i
```
<ul type="circle"><li>Listeye TR paketleri eklenir. </li></ul>

```
sudo locale-gen tr_TR
sudo locale-gen tr_TR.UTF8(TR.utf8)
```
<ul type="circle"><li>Güncelleme yapılır</li></ul>

```
sudo update-locale 
```

ya da

```
sudo apt-get update
```

<li>Dil paketi kurulduktan sonra yük ile beraber teslim edilen veri dosyası, TR dilinde oluşturulmuş veritabanına restore edilir.</li>

```
UPDATE pg_database SET datallowconn='false' WHERE datname='ATS';
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='ATS';


/* Yeni Veritabanı Oluşturma */

CREATE DATABASE "ATS" WITH OWNER=postgres ENCODING='UTF8' LC_COLLATE='tr_TR.UTF-8'
LC_CTYPE='tr_TR.UTF-8' TABLESPACE=pg_default TEMPLATE=template0;

ALTER DATABASE ATS RENAME TO ATS_OLD_15112020;

```




<h2 id="Kubernetes"></h2>

## **Kubernetes** <br/> 

<h2 id="Kubernetes_üzerinde_kurulumu"></h2>

### **Kubernetes üzerinde kurulumu**
Postgresql kubernetes üzerinde zalando/postgres-operator (https://github.com/zalando/postgres-operator) kullanarak çalışmaktadır. Bu yüzden ilk olarak postgres-operator kurulumu yapılmalıdır. 

Kurulumdan önce kurulum yapılacak olan kubernetes clusterına göre bazı ayarlamalar yapılmalıdır. Eğer veritabanı için dedike olarak worker ayrıldı ise bu workerların üzerinde sadece veritabanı koşması için taint tanımlaması yapılmalıdır. Bunun için kubernetes clusterı içerisindeki herhangi bir vm üzerinden aşağıdaki komutlar çalıştırılır.


```
kubectl taint nodes db-1 key4-value4:NoSchedule
kubectl taint nodes db-2 key5-value5:NoSchedule
kubectl taint nodes db-3 key6-value6:NoSchedule
```

Burada 'db-1' için veritabanlarının koşacağı worker'ın hostname bilgisi girilir. Key ve value bilgileri de kurulum kısmındaki yamlların içerisindeki toleration kısımları ile eşleştiği kontrol edilmelidir.

Kurulum için github üzerinde manifest klasörü üzerindeki yaml lar kullanılmaktadır. Fakat tüm yaml'lar kullanılmamakta olduğu için gerekli yamllar config olarak düzenlenmiş halde aşağıdaki dizinde bulunmaktadır.

* <u>**Burada postgres-opeartor.yml dosyası içerisindeki 'tolerations' ve 'nodeAffinty' kısımları kurulum yapılacak setup'a göre düzenlenmelidir.**</u>

```
http://zistvy90:7990/projects/A-K-V/repos/tgs2_handbooks/browse/postgresql/kurulumlar/operator
```

Bu dizin altındaki tüm yaml lar apply edilir. Daha sonra *'kubectl get pods –o wide'* ile podlar listelendiğinde aşağıdaki gibi bir postgresql-operator podunun oluştuğu teyit edilir. </br>

```
postgres-operator-7d85c849b8-4gb67     1/1     Running     0  6d18h     172.34.119.86     db3    <none> 
```

</br>
Daha sonra postgresql in verilerini tutacağı alanların oluşturulması gerekmektedir. Bu adım için postgresql pod'larının koşacağı node'lar üzerinde gerekli klasörlerin oluşturulması gerekmektedir.
</br>

```
sudo nkdir /mnt/postgres
```


<u>**Mkdir ile oluşturulan klasörlerin postgresql in koşacağı her node’da yapıldığından emin olunmalıdır.**</u> Daha sonra oluşturduğumuz klasörlere PersistentVolume oluşturulması için aşağıdaki yaml koşturulur.**<u> Burada nodeSelectorTerms altında bulunan node isimleri kısmı kurulum yapılan nodeların hostnameleri ile eşleştiğine emin olunmalıdır.</u>**

```
http://zistvy90:7990/projects/A-K-V/repos/tgs2_handbooks/browse/postgresql/kurulumlar/local-pv-volume.yaml
```


Oluşturulan PersistentVolume'ların kontrolü için aşağıdaki komut ile kontrol edilebilir.

```
kubectl get persistentvolume
```

Çıkan listede

```
pgdata-db-postgresql-cluster-0        100Gi    RwO     Retain     Available       local-postgres     25s
pgdata-db-postgresql-cluster-1        100Gi    RwO     Retain     Available       local-postgres     25s
pgdata-db-postgresql-cluster-2        100Gi    RwO     Retain     Available       local-postgres     25s

```


persistentVolume'larının olduğu ve Available durumda olduğu doğrulanmalıdır. 

PersistenVolumlar kurulduktan sonra postgresql kurulumuna geçilebilir. Bunun için 

```
http://zistvy90:7990/projects/A-K-V/repos/tgs2_handbooks/browse/postgresql/kurulumlar/minimal-postgres-manifest.yaml
```

yamlı apply edilir. Burada postgresql lerin oluşturduğumuz persistentVolume lara bind olması beklendiğinden yaml üzerinde herhangi bir node ismi belirtilmesi yapılmamıştır. Sadece 

Daha sonra podlar konrol edildiğinde aşağıdaki Resim 5-2 olduğu gibi podların 'Running' durumda olduğu teyit edilir.
```
netas@atsl:~$ kubectl get pods -o wide
NAME                               READY        STATUS      RESTARTS        AGE         IP                NODE
akhq-ff869d5dc-99nnq               1/1          Running     6               9d          172.34.177.153    ats2
db-postgresql-cluster-0            2/2          Running     0               3d2h        172.34.119.101    db3
db-postgresql-cluster-1            2/2          Running     0               82s         172.34.151.3      db1
db-postgresql-cluster-2            2/2          Running     0               3d1h        172.34.233.3      db2
```
<h2 id="Kubernetes_uzerinde_baglanti_bilgilerinin_edinilmesi"></h2>

### **Kubernetes üzerinde bağlantı bilgilerinin edinilmesi** <br/>
Kubernetes üzerinde Postgresql kurulumunda pgadmin üzerinden bağlantı sağlanılması için port ve password bilgilerine ihtiyaç duyulmaktadır. Port bilgisi için db-postgresql-cluster isimli kubernetes servisinden yararlanılmaktadır. Bu servis ile postgresql kubernetes dışında da erişilebilir olmaktadır.

```
netas@atsl:~$ kubectl get services
NAME                            TYPE             CLUSTER-IP               EXTERNAL-IP         PORT(S)             AGE
akhq                            NodePort         10.100.94.55             <none>              80:31379/TCP        9d
db-postgresql -cluster          LoadBalancer     10.106.69.159            <pending>           5432:31147/TCP      4d2h
db-postgresql-cluster-config    ClusterIP        None                     <none>              <none>              4d2h
db-postgresql-cluster-repl      LoadBalancer     10.105.160.30            <pending>           5432:30853/TCP      4d2h
```



Resim 5-3 de görüldüğü üzere 'db-postgresql-cluster' için dışarıya açılan port bilgisinin '31147' olarak tahsis edildiği öğrenilebilir. <u>**Bu port bilgisi her postgresql kurulumunda değişmektedir.**</u> Bu yüzden her kurulum sonrası port bilgisi tekrar öğrenilmedilir. Ayrıca 'db-postgresql-cluster-repl' servisi ile postgresql’in replica podlarına da ulaşılabilir.

Postgres user'ının password bilgisi kubernetes secret dosyaları üzerinde muhafaza edilmektedir. Bu bilgi aşağıdaki komut ile edilinebilir. 

```
kubectl get secret postgres.db-postgresql-cluster.credentials -o 'jsonpath={.data.password}' | base64 -d
```


Yukarıdaki komut ile postgres kullanıcısının password bilgisi edilinebilir. <u>**Burada password bilgisi her postgresql kurulumundan sonra değişmektedir.**</u>

<h2 id="Faydali_linkler"></h2>

# **FAYDALI LİNKLER**
<li>Veritabanı Taslak Tabloları Vob Bağlantı Linki:</li>

[M:\main_merge\ddxadm\ddxadm\document_library\Systems\CBTC_ATS_GYH\DESIGN\SERVICES\Command_Control_Service\Veritabani_Yonetimi\Veritabani_Tablolari](M:\main_merge\ddxadm\ddxadm\document_library\Systems\CBTC_ATS_GYH\DESIGN\SERVICES\Command_Control_Service\Veritabani_Yonetimi\Veritabani_Tablolari)


<li>ER Diyagramını görüntülemek için VS Code üzerine ERD Editör eklentisini kurulması gerekmektedir:</li>

[https://marketplace.visualstudio.com/items?itemName=dineug.vuerd-vscode](https://marketplace.visualstudio.com/items?itemName=dineug.vuerd-vscode)


<li>Zalando postgres operatör: </li>

[https://github.com/zalando/postgres-operator](https://github.com/zalando/postgres-operator)

<h2 id="Tablolar"></h2>

# **TABLOLAR**

    1	alarm_confirm_logs
    2	alarm_descriptions
    3	alarm_logs
    4	announcement_systems_logs
    5	areas_of_dispatch
    6	areas_of_parking
    7	ats_constants
    8	ats_service_logs
    9	auth_categories
    10	auth_sub_categories
    11	auths
    12	cesb_logs
    13	cmr_logs
    14	commands
    15	commands_causes
    16	config_logs
    17	connection_logs
    18	contactor_logs
    19	daky_languages
    20	daky_logs
    21	demands
    22	door_logs
    23	energy_area_logs
    24	eqp_announcement_systems
    25	eqp_ats_services
    26	eqp_cesbs
    27	eqp_cmrs
    28	eqp_connections
    29	eqp_contactors
    30	eqp_doors
    31	eqp_energy_areas
    32	eqp_feis
    33	eqp_interface_relays
    34	eqp_maintenance_logs
    35	eqp_map
    36	eqp_midas
    37	eqp_midas_channels
    38	eqp_midas_sensors
    39	eqp_obatc_connections
    40	eqp_occs
    41	eqp_pakses
    42	eqp_pesbs
    43	eqp_platforms
    44	eqp_plcs
    45	eqp_point_controllers
    46	eqp_pos
    47	eqp_radios
    48	eqp_rios
    49	eqp_routes
    50	eqp_servers
    51	eqp_stations
    52	eqp_switches
    53	eqp_tracks
    54	eqp_trains
    55	eqp_utcs
    56	eqp_virtual_machines
    57	eqp_virtual_signals
    58	eqp_workstations
    59	eqp_wsatc_connections
    60	eqp_wsatp
    61	eqp_wsaux
    62	eqp_ybs
    63	equipments_pesb
    64	equipments_table_names
    65	event_logs
    66	fei_logs
    67	hay_configs
    68	interface_relay_logs
    69	locations
    70	loop_dwells
    71	loops
    72	mai_closed
    73	mai_conditions
    74	mai_constants
    75	mai_counters
    76	mai_list
    77	map_logs
    78	midas_channel_logs
    79	midas_logs
    80	midas_sensor_logs
    81	occ_logs
    82	old_table
    83	operator_topic_list
    84	paks_logs
    85	pesb_logs
    86	platform_logs
    87	plc_logs
    88	point_controller_logs
    89	pos_logs
    90	radios_logs
    91	report_filter
    92	rios_logs
    93	role_auth
    94	role_session
    95	role_work_station
    96	role_zone
    97	roles
    98	route_group_logs
    99	route_logs
    100	selectable_filters
    101	servers_logs
    102	stations_logs
    103	switch_logs
    104	timetable_constraint_platforms
    105	timetable_constraint_speeds
    106	timetable_infos
    107	timetable_loops
    108	timetable_periodic_exceptions
    109	timetable_periodics
    110	timetable_plan_history
    111	timetable_plans
    112	timetables
    113	track_logs
    114	tracks
    115	train_dwell_time_logs
    116	train_footprints
    117	train_graph_reports
    118	train_logs
    119	train_ma
    120	train_on_board_logs
    121	train_performance_logs
    122	train_performances
    123	train_trip_info_logs
    124	train_virt_occups
    125	user_logs
    126	users
    127	utcs_logs
    128	virtual_machines_logs
    129	virtual_signal_logs
    130	workstations_logs
    131	wsatc_connections_logs
    132	wsatp_logs
    133	wsaux_logs
    134	ybs_logs
    135	zones


</body>
</html>