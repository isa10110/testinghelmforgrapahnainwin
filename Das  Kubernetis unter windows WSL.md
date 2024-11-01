# The Kubernetis under windows WSL: 
Idea: i would like to try to configure my windows so that i have a local cluster for the different spaces of my kubectl and once the desy Dchace rencher cluster. for this i would like to emulate single hosts by WSL ... 
realization it does not work (without further ado)

## The Kubernetis under windows WSL: Setting up our tools:

About this tutorial: 
I have linux user experience and I am not familiar with many of the Windows features.  Therefore I will give some explanations that I have learned here in the side notes / quotes. 

In this tutorial I will explain why I did certain things and why they were necessary. 

The tutorial will be limited to 3 parts Systemconfiguratin 
Kubernetis in general 
and the goal to create Helm Charts for Graphana.  

### Insatall WSL: (instances) 
first teps

1. installing WSL  

on the cli (powershell) : 

````
wsl --install
````

2. choose your Distro

 Ubuntu  WSL  take out of the Software Store ... 
 Debian WSL  take out of the Software Store  ... 

> The Mikrosoft store is a showroom, not a package manager, so the installed software must be managed and updated externally.

3. configure WSL: 

chainging the Hostname on your wsl.
for ubuntu follow this tuturial , wsl uses sytemd but implement features diffrent than expected.
https://medium.com/@AnupamMajhi/change-ubuntu-hostname-running-on-wsl-7122b83fd6ed 

>this Tutorial  only works for ubuntu,  the Debian WSL build differently
> the standard Systemd Tools for chainging the  the hostnameanzeige funkion some howin  debian (the chainges would take over for two  boots than be removed ) 

Install  HSTR
https://github.com/dvorka/hstr/blob/master/INSTALLATION.md#ubuntu

Konfigurieren der Bashrc mitl hstr

HSTR is a tool that makes your bash history serchable,

(Everyone has their own preference here HSTR is a very convenient tool to use the bashrc)

> set up necessary alias functions, (it is advisable to enter a cfetch or neofetch at the end of the config so that you can quickly recognize which wsl you are in when logging into the bash. )

>normal cli systhem tools like htop do not need to be installed, as they are not necessarily mapped to the hardware correctly. currently you can cover what htop provides with **top** and **free** to cover all implied features to a large extent. 
>
>(htop still has some advantages for the display, see htop config, but in our use case you come less into contact with it)

Install Standard Software
```
sudo apt update; sudo apt upgrade -y; sudo apt autoremove; sudo apt install  vim, neovim, neofetch
```

tools can be changed by preference :) 

als nächstes müssen wir podman Installieren 

> Docker recognizes that we are in a WSL and would rather recommend Docker for windows. 

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
type  **minikube version** to test minikube. 

start Minikube

you can do the following step as root 

```
sudo minikube start --force
```
minicube doesn't like the fact that minicube is already running a vm, so forcing it causes it to start anyway.

```
minikube status
```
now you can install kubectl  follow this official guidlines.
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

you  want to use the cluster from external therefore we have to add ouer user to  the sudoers group 

```
sudo usermod -a -G $user
```

then you  have to  ad your  minicube config to run as non root 

````
 minikube config set rootless true
````

we now bend the ip  to  0.0.0.0 therefore we use the minikube  option --listen-address='0.0.0.0'

This is a potential  securaty risc  but it shuld allow us to  acsess the ip  of wsl  system .

(and it dont works therefore i shuld dig into that difrently it dont work in userspace run it  as root  dos  the trick **sudo minikube start --force**)

futher investigation could be done :)



Based on this tutorills
https://medium.com/@areesmoon/installing-minikube-on-ubuntu-20-04-lts-focal-fossa-b10fad9d0511
https://medium.com/@redswitches/how-to-install-podman-on-ubuntu-20-04-and-22-04-81592ef0e3d1
https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fdebian+package
and some more googleling

### Setup Windows as client kubectl client 

we want to get the two kubernetes instances under windows. 
There are 2 essential tools for this 

Kubectl and helm.  Kubectl is an important tool to manage a kubernetes cluster, to start and stop pods that can consist of one or more containers, to configure the pods with deployments that get interfaces to another, for example via services and config maps. 

these configurations are templated by helm chart so that simpler kube configuration is possible.  Helm chart manages many configurations for kubernetis, such as a package manager like apt. or later winget.

#### Setting up kubectl under windows.

windows terminals are different from what we know from the linux world,
To summarize: 
windows has a CMD, a Powershell and a Powershell ISE, all these have their own peculiarities, we will use the Powershell in this tutorial. 

>your powershell runs in a terminal emulator, normal control +shift +c and control+shift+v will not work if you nerf it install Alacritty https://github.com/alacritty/alacritty this will make it feel a bit more like a standard terminal emulator.
>you  would also get in touch with msys2  its allso a good oportonity to  do linux stuff un windows but its more pain theen wsl  in some cases it adds their own cli ,  you  will got in touch if you  install  git for windows xD

install  kubectl

``` 
winget install -e --id Kubernetes.kubectl
```
now you have to put the configuration of your kubernetis cluster into the .kube in your user directory. 
To do this, you must create the folder and copy the file you receive in the ranshers of your kubernetis cluster into the directory with the name config.

> windows toolig also  gui  stuff is strange sometimes anything works tiffrent as expected. for example if you want to create a folder you cant do it propaly using the gui you are better of using the unixi  like  terminal. mkdir ls touch be aware that these tools differ in funktion from cmd to  powershell

add into  the local kubeconfig file the following lin into the cluster config namespace dependent.
```
namespace: graphana 
```
set a default namespace read more about rbac :) 

this line gives the default namespace.

in der yaml  sektion  context: unter contexts: hinzu: 

> there is no difference  between .yml  and .yaml files. theire are systems that struggle with a 4 signe datasuffix.

for futher clusters the conficuration can be placed at the (**contexts**) section

than you  can install  helm chart for your kubectl

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

>many things you  normaly do in the cli  you  can now do by hand :) 



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

the  grphana is changed that it can run with out the right to modyfing  the  namespace its the same fpr prometheus , the same is for the prometeus char.  prometeus has an option to  place on any node a automatic collector  for metricis,  I disabled it for leake of pemissions 

(configurable in .\prometheus.yaml) line 1286

you have now conect both  sevices maualy. 
this could be possibly solfed with  a  helmcaht that wraps other charts. 

>helm upgradel my-grafana grafana/grafana -f grafanaconf.yaml

with the related upgrade kommands you can upgrade the chart by edding the config

we can combine both by creating a subchart 
https://medium.com/craftech/one-chart-to-rule-them-all-3f685e0f25a9

it possibly the best to do it where we want to ship that.


https://grafana.com/docs/grafana/latest/setup-grafana/installation/helm/
https://github.com/prometheus-community/helm-charts
