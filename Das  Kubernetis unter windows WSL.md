# Das  Kubernetis unter windows WSL: 
Idee: ich würde gerne versuchen mein windows so  zu konfigurieren das ich den verschiedenen spaces meines kubectls  ein locales cluster und einmal das desy dchace rencher cluster habe. dafür würde ich gerne einzelene hosts durch WSL  emuliern... 
Erkenntnis es funktioniert nicht (ohne weiteres)

## Einrichtung unserer Tools: 
Zu  diesem Tutorial: 
Ich habe c.a 10  jahre linux nutzer Erfahrung und bin mit vielen geflogenheiten unter windows nicht vertraut.  Daher werde ich hier auch vieleicht einige erklärungen dalegen die ich hier gelernt habe in den seitnotizen / zitaten. 

in diesem Tutorial werde ich dalegen wiso ich bestimmte dinge getan  habe,  und wiso diese notwendig waren. 

Das tutorial wird sich auf 3  teile beschrenken Systemconfiguratin 
Kübernetis im Allgemeninen 
und das Ziel ein Helm Chart für Graphana zu erstellen.  

### Einrichtung WSL: (Instanzen) 
erste schritte: 

1. installing WSL  

in der cmd : 

````
wsl --install
````

2. schritt 

 Ubuntu  WSL  aus dem Software Store holen ... 
 Debian WSL aus dem Software Store Holen ... 

> Der Mikrosoft store ist ein showroom kein paket mannager,  Daher muss die installierte software extern verwaltet und Aktualisiert werden.

3. WSL konfigurieren: 

Ändern  des Hostnamens
folge für ubuntu dem tuturial , das ist gut denn wsl macht dinge anders als sytemd es vorsieht.
https://medium.com/@AnupamMajhi/change-ubuntu-hostname-running-on-wsl-7122b83fd6ed 
>dieses Tutorial  funktioniert nur für ubuntu,  das Debian WSL ist anders gebaut
> Die standard Systemd Tools für die hostnameanzeige funkioniern halb im debian container (die verenderungen werden nicht persistent übernommen) und ganicht im ubutu container 

Installieren von HSTR
https://github.com/dvorka/hstr/blob/master/INSTALLATION.md#ubuntu

Konfigurieren der Bashrc mitl hstr
(Jeder hat hier seine eigene präferenz  HSTR ist ein sehr konfortables tool  um die bashrc zu nutzen)

> einrichten notwendiger alias funkttionen,  (es ist ratsam am ende der konfig ein cfetch oder neofetch einzutragen damit man beim login in die bash schell erkennen kann in welchem wsl man ist. )

>normale cli systhem tools wie htop müssen nicht instlliert werden,  da sie nicht unbedingt zutreffend auf die hardware gemappt werden  derzeit kann man das was htop liefert mit **top** und **free** abdecken um alle implimentierten features weitgehend abzudeken. 
(htop  hat noch einige vorteile für die darstellung siehe htop config, jedoch kommt man in unserm usecase weniger damit in berührung)

Installieren standard Software
```
sudo apt update; sudo apt upgrade -y; sudo apt autoremove; sudo apt install  vim, neovim, neofetch
```

tools können nach vorliebe variirt werden.

als nächstes müssen wir podman Installieren 

> Docker erkennt das wir uns in einem WSL Befinden und Möchte uns lieber Docker for windows ans hertz legen. 

```
sudo apt -y install podman
```
now enable podman 

```
sudo  systemctl start podman 

sudo  systemctl enable podman
```

you  can inform yourself if it worked by the **status** option of systemctl

now you  can  logout and test podman  mit  **podman -v** 


this are the tools you need for the minicube installation
```
sudo apt install conntrack coreutils  curl wget apt-transport-https
```
Downlod the installer
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
install minikube

```
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
type  ** minikube version** to test minikube. 

start Minikube

you can do the following step as root 

```
sudo minikube start --force
```
das das minicube  schon einer vm  läuft mag minicube nicht so gerne das forcen führt dazu das das trotzdem started.

```
minikube status
```
now you can install kubectl  follow this official guidlines.
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

you  want to use the cluster from external therefore we have to add ouer user to  the sudoers group 

```
sudo usermod -a -G $user
```

then you  have to  add your  minicube config to run as non root 

````
 minikube config set rootless true
````

we now bend the ip  to  0.0.0.0 therefore we use the minikube  option --listen-address='0.0.0.0'

This is a potential  securaty risc  but it shuld allow us to  acsess the ip  of wsl  system .

(and it dont works therefore i shuld dig into that difrently it dont work in userspace run it  as root  dos  the trick **sudo minikube start --force**)



Based on this tutorills
https://medium.com/@areesmoon/installing-minikube-on-ubuntu-20-04-lts-focal-fossa-b10fad9d0511
https://medium.com/@redswitches/how-to-install-podman-on-ubuntu-20-04-and-22-04-81592ef0e3d1
https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fdebian+package
and some more googleling

ggf die Bash  rc weiter anpassen. 

### Einrichtung Windows als Client kubectl client 

wir wollen unter windows die beiden kubernetes instanzen erhalten. 
dafür giebt es 2 essenzielle tools 

Kubectl und helm.  Kubectl  ist ein wichtiges tool um ein kubernetis cluster zu managen,  Pods die aus ein oder mehreren kontainern bestehn können zu  starten ud zu stoppen,  die Pods zu konfiguriern mit duch deployments,  die zu ein ander schnittstellen bekommen zum beispile über services und konfig maps. 

diese konfigurationen  werden durch helm chart getemplated sodass einfachere kube konfiguration möglich werden.  Helm chart verwaltet viele konfigurationen für kubernetis ,  wie ein paketmannager wie apt. oder später winget.

#### Einrichtung von kubectl unter windows. 

windows terminals sind anders als wir das aus der linux welt kennen,
unm   das zusammenzufassen: 
windows hat eine CMD,  eine Powershell und eine Powerschell ISE,  All  diese haben ihre eigenen eigenheiten, wir werden in diesem tutorial die Powershell verwenden. 

>ihre powershell läuft in einem terminal  emulator, normales control +shift +c und control+shift+v funktionirt nicht wenn sie das nerft  installieren sie sich  Alacritty https://github.com/alacritty/alacritty  damit fühlt sich der kram dann ein wenig mehr wie ein standard terminal emulator an.
> you  would also get in touch with msys2  its allso a good oportonity to  do linux stuff un windows but its more pain theen wsl  in some cases it adds their own cli ,  you  will got in touch if you  install  git for windows xD

erste installation von Kubernetes kubectl

``` 
winget install -e --id Kubernetes.kubectl
```
nun müssen sie die konfiguration ihres kubernetisclusters in das  .kube in ihrem nutzer direktory legen. 
hierfür müssen sie den ordner erstellen und das file was sie in dem ranshers ihrer kübernetis cluster erhalten in das direktory kopieren mit dem  namen config.

> windows toolig also  gui  stuff ist seltsam teilweise endstehen ordnernamen mit seltsamen zeichen darin am sichersten genen sie wenn sie das mit den standard unix bordmitteln machen. mkdir ls touch (diese funktioniern in der Powershell cmd ist teilweise anders ...)

in der cubeconfig fügen sie am besten mit die zeile 
```
namespace: graphana 
```
set a default namespace read more about rbac :) 

diese zeile setzt den default namespace in dem kontext 

in der yaml  sektion  context: unter contexts: hinzu: 

> Es giebt keinen unterschied zwischn .yml und .yaml files es giebt wohl ältere  systeme die keine 4 zeichen dateisuffix vertragen.

weitere cluster configuration würden so auch hier eingefügt (**contexts**)

im  anshluss bitte Helm  Chat installieren das sollte die konfiguration von kubectl verwenden. 

```
winget install Helm.Helm
```
Based on:
https://helm.sh/docs/intro/install/
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
many other futer resouces 

> side info  we  have DSM too,  with winget you  go in the rigth direction ^^

now you  can try to  get recoures from kubernetis 
if your rights are not enough like what you are willing to do you  now have two  options,  find a  way around or grant this privilages ... 
i was into  the first aproge caus we only have to  grant privilages for nesseserry things right ...  therrefore we shuld change a bit 

Use full  commands 

kubectl  get pods 
kubectl get all  

>> many things you  normaly do in the cli  you  can now do by hand :) 



we have to add reposetorys that we can use helm  carts for graphana and phrometeus 

> helm repo add grafana https://grafana.github.io/helm-charts
> helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

than you want to know if enythink worked 

> helm repo list

now you shuld see at least two  repos :)

grafana and prometheus-community

than you can giv the following commands
> helm install monitoring-prometheus prometheus-community/prometheus -f .\prometheus.yaml
>  helm install my-grafana grafana/grafana -f grafanaconf.yaml

now we have two  sporned up  configs ...  the  promometeus and graphanaconf  are based on the original  valuses files with changed settings 

the  grphana is changed that it can run with out the right to modyfing  the  namespace its the same fpr prometheus , the same is for the prometeus char.  prometeus has an option to  place on any node a automatic collector  for metricis ,  I disabled it for leake of pemissions 

you have now conecct both  sevices maualy. 



https://grafana.com/docs/grafana/latest/setup-grafana/installation/helm/
