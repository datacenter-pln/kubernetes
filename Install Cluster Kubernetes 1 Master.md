Cara Install Kubernetes di Linux Ubuntu
Mau install kubernetes di Linux/Debian based? Atau mau belajar kubernetes di Linux Ubuntu… ikuti langkah-langkahnya:
Berikut spesifikasi yang dibutuhkan
1.	Server berbasis Linux
2.	Minimal 2 server . Dimana 1 server digunakan sebagai master, dan yang satunya lagi sebagai minion/worker node.
Minimum requirement :
•	CPU core min  +2 core.
•	Ram minimum 1gb (Untuk keperluan production harus diatas 8gb yah guys! 1 gb ini gpp kalau cuma dipakai buat belajar   tapi tidak untuk production nanti)
•	Linux tanpa partisi swap (kubernetes tidak bisa diinstall jika system memiliki swap)
Sebelum memulai installasi dan setup, lakukan berikut ini terlebih dahulu:
Lakukan update dan upgrade system
•	sudo apt-get update && sudo apt-get upgrade
Agar terlihat rapi kita akan memberi nama/hostname Master => svr1-ubu-k8s-master dan Minion/worker sebagai svr1-ubu-k8s-minion1.
Penamaan hostname kalau bisa unik yah, contoh jika kita ingin buat server minion baru pada cluster yang sama maka kita bisa kasih nama svr1-ubu-k8s-minion2.
Master
•	sudo hostnamectl set-hostname svr1-ubu-k8s-master
Ubah file /etc/hosts dan masukan baris line svr1-ubu-k8s-master disebelah kanan 127.0.0.1 sehingga menjadi
 
Minion/Worker
•	sudo hostnamectl set-hostname svr1-ubu-k8s-minion1
Ubah file /etc/hosts dan masukan baris line svr1-ubu-k8s-minion1 disebelah kanan 127.0.0.1 sehingga menjadi
 

Hapus Swap
swapoff -a
nano /etc/fstab
 
Memulai: Cara Install Kubernetes dan Docker di Linux Server
(Lakukan cara berikut ini di master dan juga minion)
sudo apt-get install apt-transport-https -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
Buat yg ngalamin error :
gpg: can’t open ‘–’: No such file or directory tolong ganti add – pada baris paling akhir  dengan -(dash yah) wordpressnya reseh autocorrect sendiri. mengubah -menjadi double–)
Jalankan perintah selanjutnya:
sudo nano /etc/apt/sources.list.d/kubernetes.list
Masukkan baris text berikut:
deb http://apt.kubernetes.io/ kubernetes-xenial main
Tekan CTRL+X untuk simpan. Lalu lanjut jalankan perintah ini untuk update repo:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
sudo reboot
Setup dan Konfigurasi (Pastikan semua node sudah kamu reboot dahulu sebelum memulai langkah lanjutan ini)
Master Only (Jalankan perintah ini hanya di Master)
sudo kubeadm init --pod-network-cidr 10.244.0.0/16
Setelah menjalankan proses ini akan muncul pesan pada bagian bawah. Mohon dicatat dan disimpan. Terutama baris line seperti dibawah ini karena akan kita digunakan jika ingin menambah minion-minion:
You can now join any number of machines by running the following on each node as root: kubeadm join 172.31.3.157:6443 –token f31esk.s9e5sib94vr9elqi –discovery-token-ca-cert-hash sha256:610a3349a591a07f671abfd9bd262ccb40e26fd91e4db8509
Lanjut jalankan ini di Master.
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
Setup POD
Pod adalah unit atau boleh dibilang adalah wadah kubernetes dalam menyimpan container docker. Umumnya 1 pod berisi satu container walau sebenarnya bisa juga 1 pod berisi banyak container.
Jalankan perintah ini hanya di Master
 
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
Jalankan perintah ini lagi di master untuk memastikan keseluruhan pod terdeploy dan running dengan baik.
sudo kubectl get pods --all-namespaces
 
Perintah diatas adalah untuk melakukan deploy flanel, dimana functionnya kurang lebih sebagai networking dan deploy heapster dimana functionnya sebagai metric server disisi kubernetes.
Sebelum lanjut ke langkah join minion pastikan kamu sudah melakukan langkah2 di kategori: Memulai: Cara Install Kubernetes dan Docker di Linux Server (Lakukan cara berikut ini di master dan juga minion)
________________________________________
JOIN MINION
Minion Only (Jalankan perintah ini hanya di Minion)
Jalankan sebagai root atau sudo untuk join minion ke master:
kubeadm join 172.31.3.157:6443 –token f31esk.s9e5sib94vr9elqi –discovery-token-ca-cert-hash sha256:610a3349a591a07f671abfdb9bd262ccb40e26fd91e4db8509
Kembali ke terminal master, coba cek dengan perintah ini:
•	
o	
	kubectl get nodes
Ulangi beberapa kali perintah diatas (kubectl get nodes) sampai status master dan minion menjadi READY (dibutukan waktu beberapa detik/menit untuk status menjadi ready).
 
ROLES minion masih tertulis NONE. Untuk memberi label maka jalankan perintah ini:
kubectl label node svr1-ubu-k8s-minion1 node-role.kubernetes.io/minion1=
 
________________________________________
Deploy Kubernetes Dashboard (Control Panel) (Jalankan perintah ini hanya di master)
sudo kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
sudo kubectl get pods --all-namespaces
kubectl -n kube-system get service kubernetes-dashboard
Kita tidak bisa langsung akses dashboard kubernetes. Karena semua container kubernetes dan clusternya hanya bisa diakses internal. Kita perlu mensetup network disisi router dll agar ada skema forwarding dan sejenisnya sehingga bisa diakses dari luar/public.
Untuk langkah sederhananya kita juga bisa pasang VPN di server master sehingga kita bisa akses dashboard menggunakan VPN. Dengan alamat: https://IPCluster
Untuk mengetahui IP cluster dashboard gunakan cara ini:
•	
o	
	kubectl -n kube-system get service kubernetes-dashboard
 
ALTERNATIF LAIN (Gunakan cara ini jika tidak menggunakan skema diatas) :
Akses Dasboard langsung tanpa vpn (Tidak disarankan untuk production, cara ini digunakan hanya untuk keperluan testing/development)
Kita  bisa melakukan perubahan berikut agar bisa diakses dari luar namun untuk production tidak disarankan yah. Jika production mohon gunakan skema networking yang baik atau via VPN.
kubectl -n kube-system edit service kubernetes-dashboard
Ubah 1 kata paling bawah yaitu: ClusterIP menjadi NodePort. Lalu simpan
 
kubectl -n kube-system get service kubernetes-dashboard
Akan muncul port node expose. maka kita bisa akses Kubernetes dasboard menggunakan port tersebut.
 
Maka kita bisa langsung akses dibrowser dengan alamat https://IPServer:port
Contoh pada gambar diatas maka diakses dengan alamat https://IPServer:31660. (Note: Buat yg akses dashboard ada error certificate ssl silahkan buka melalui firefox. ;))
Ubah IP server menjadi IP server kita (jangan akses menggunakan cluster IP (10.100.18.68 pada gambar diatas.)
________________________________________
Membuat akses Login Kubernetes Dashboard
Jika kamu telah berhasil setup dashboard kubernetes maka langkah selanjutnya adalah membuat user untuk dapat login ke dashboard kubernetes. Berikut caranya.
Masih di terminal server Master ketik perintah berikut:
$ kubectl create serviceaccount cluster-admin-dashboard-sa
$ kubectl create clusterrolebinding cluster-admin-dashboard-sa \
--clusterrole=cluster-admin \
--serviceaccount=default:cluster-admin-dashboard-sa
 
$ kubectl get secret | grep cluster-admin-dashboard-sa
$ kubectl describe secret cluster-admin-dashboard-sa-token-x6gh7
Akan muncul token yang bisa digunakan sebagai login. Silahkan login menggunakan token tersebut, masukan pada dashboard login kubernetes.
 
 
 
 
Sumber https://www.ayies.com/cara-install-kubernetes-di-linux-ubuntu/

