# Storage

## NFS - Ağ Dosya Sistemi
- Dosyalara ve sunucu dosya sistemlerine şeffaf erişim sağlar. Herhangi bir istemcinin yerel bir dosyayla çalışabilen uygulamasının, herhangi bir program değişikliği yapmadan NFS dosyasıyla çalışmasını sağlar.

## Ceph Depolama Kümesi
- Ceph, tek bir platformda Object Storage, Block Storage ve File System sağlama gereksiniminin en iyi şekilde karşılanan gelişmiş ve ölçeklenebilir bir Software-defined storage'dir.
- Kübernet bölmelerinde kalıcı depolama için CephFS veya CephRBD'yi kullanılır.
- Ceph RBD, kapsüle atadığımız blok deposudur. CephRBD okuma-yazma modunda aynı anda iki bölme ile paylaşılamaz.
- CephFS, verileri en üst Ceph kümesinde depolayan POSIX uyumlu bir dosya sistemi hizmetidir. CephFS'yi aynı anda birden çok kapsülle paylaşabiliriz.

## GlusterFS Depolama Kümesi
- GlusterFS, bulut depolama için uygun ölçeklenebilir bir ağ dosya sistemidir. Aynı zamanda Ceph gibi commodity donanımında çalışan yazılım tanımlı bir depolama alanıdır, ancak sadece Dosya sistemleri sağlar ve CephFS'ye benzer.
- Glusterfs, ceph'e kıyasla daha büyük blok boyutu kullandığından Ceph'den daha fazla hız sağlar, yani Glusterfs 128kb'lik bir blok boyutu kullanırken ceph 64Kb'lik bir blok boyutu kullanır.

## AWS EBS Blok Depolama
- Amazon EBS, EC2 bulut sunucularına bağlı kalıcı blok depolama birimleri sağlar. AWS, EBS için çeşitli seçenekler sunar, bu nedenle IOPS sayısı, depolama türü (SSD / HDD) vb. Gibi parametrelere bağlı olarak depolamayı gereksinime göre seçebiliriz.
- AWSE EBS'yi AWSElasticBlockStore kullanarak kalıcı blok depolama için kubernetes bölmeleriyle monte ediyoruz. EBS diskleri, dayanıklılık ve yüksek kullanılabilirlik için otomatik olarak birden fazla AZ'ye kopyalanır.

## GCEPersistentDisk 
- Google Cloud Platform ile kullanılan dayanıklı ve yüksek performanslı bir blok depolama alanıdır. Google Compute Engine veya Google Container Engine ile kullanabiliriz.
- HDD veya SSD arasından seçim yapabiliriz ve ihtiyaç arttıkça birim diskin boyutunu artırabiliriz. GCEPersistentDisks, dayanıklılık ve yüksek kullanılabilirlik için otomatik olarak birden fazla veri merkezinde çoğaltılır.

## Azure Disk 
- AWS EBS ve GCEPersistentDisk gibi dayanıklı ve yüksek performanslı bir blok depolama alanıdır. Ortamınız için SSD veya HDD arasından seçim yapma seçeneği ve Zamanında yedekleme, kolay taşıma gibi özellikler sunar.
- Azure Veri Diski'ni bir Pod'a bağlamak için bir AzureDiskVolume kullanılır. Azure Diskleri, yüksek kullanılabilirlik ve dayanıklılık için birden çok veri merkezinde çoğaltılır.


|Storage Technologies|Read Write Once|Read Only Many|Read Write Many|Deployed On|Internal Provisioner|Format|Provider|Scalability Capacity Per Disk|Network Intensive|
|---|---|---|---|---|---|---|---|---|---|
|CephFS| Yes| Yes|Yes|On-Premises/Cloud|No| File| Ceph Cluster|Upto Petabytes|Yes|
|CephRBD| Yes| Yes|No|On-Premises/Cloud|Yes|Block|Ceph Cluster|Upto Petabytes|Yes|
|GlusterFS| Yes| Yes|Yes|On-Premises/Cloud|Yes|File| GlusterFS Cluster| Upto Petabytes|Yes|
|AWS EBS| Yes| No |No|AWS|Yes|Block|AWS|16TB|No|
|Azure Disk| Yes| No|No| Azure|Yes|Block| Azure|4TB|No|
|GCE Persistent Storage| Yes| No | No|Google Cloud|Yes|Block|Google Cloud|64TB/4TB (Local SSD)|No|
|OpenEBS| Yes| Yes|No| On-Premises/Cloud| Yes|Block| OpenEBS Cluster| Depends on the Underlying Disk|Yes|
|NFS| Yes| Yes | Yes|On-Premises/Cloud|No|File|NFS Server|Depends on the Shared Disk|Yes|