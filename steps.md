https://www.youtube.com/watch?v=FTrTFOLbdm4&t=1666s
The Ultimate CICD Corporate DevOps Pipeline Project | Real-Time DevOps Project | GitHub Actions

Dis adv:
platform dependent (GitHub - azure, aws commit)
limited plugins


Adv:
can check the code on different versions
can create new self hosted-runners/similar to docker image for agent in Jenkins (but docker image runs on separate host in Jenkins)
opensource project/public project/not moving from one place to another
no need to install too many tools (java 17, SonarQube scanner, SonarQube quality gate check)

k8-1 / worker-1 (t2.med)

sudo apt update
follow k8s-setup file
kubectl get nodes (1-master/1-worker)

Runner / t2.large (VM to run all commands)
sudo apt update
github/settings/actions/runners/new self-hosted runner/linux/
download: 2 sets of commands - copy n paste
next tar -xvf actions-runner-linux-x64-2.321.0.tar.gz

configure: 1st command / enter/dev-runner/skip label (runner succesfully added)
again goto github/settings/actions/runners - runners created, but offline
ls
./run.sh
refresh page (idle)

in Runner (ec2)/setup sonarqube server
install docker: sudo apt install docker.io
sudo chmod 666 /var/run/docker.sock (for other users to use docker commands without sudo)
docker run -itd -p 9000:9000 sonarqube:lts-community
chrome: publicIP:9000 (admin/admin change pwd)
install maven: sudo apt install maven

github/actions/new workflow/select java with maven/configure
automatically creates /.github/workflows (name pipeline of your wish 333.yml
(modify pipeline)
name: CICD Pipeline
(delete pull, which means any changes to push, pipeline will be triggered)
runs-on: self-hosted
run: mvn package --file pom.xml
commit changes (in the repo, u'll find a brown dot, click it, pipeline is in progress)

Install trivy - 5 steps in Runner
trivy --version

goto repo/.github/workflows/333.yml

write workflow file

GitHub/settings/secrets and variables/secrets/new repository secret
name: SONAR_TOKEN
paste token 
(goto SonarQube/administration/security/users/create token/copy pwd)

name:SONAR_HOST_URL
http://13.234.226.74:9000

DOCKERHUB_USERNAME

DOCKERHUB_TOKEN (pwd)

in k8s-master:

create svc, role, role binding and secrets in k8s master

kubectl create namespace webapps
vi svc.yml
kubectl apply -f svc.yml
vi role.yml
kubectl apply -f role.yml
vi bind.yml
kubectl apply -f bind.yml (change name: git-actions-svc)
vi secret.yml (change name: git-actions-svc-secret // kuber..: git-actions-svc)
kubectl apply -f secret.yml

cd .kube/
ls
cat config
complex (do not work)
base64 ~/.kube/config
copy (and paste in below)

GitHub/settings/secrets and variables/secrets/new repository secret
name: KUBE_CONFIG
paste token 

change image in deployment-service.yaml

commit changes

since we are not using AWS, no external IP is present
since only 1 worker node, chrome: publicIP:nodeport
((kubectl get svc -n webapps, copy nodeport))

EC2: monitoring (t2.large)

sudo apt update

chrome: prometheus.io/download/prometheus for linux
wget (url link)/ls
tar -xvf <tar file> (to unzip)

cd prometheus
./prometheus & (&, so that it runs in background)
enter

chrome: publicIP of runner:9090 (for prometheus)

chrome: prometheus.io/download/blackbox exporter for linux
wget (url link)/ls
tar -xvf <tar file> (to unzip)

cd ..
cd blackbox
./blackbox_exporter &
enter

chrome: publicIP:9115 (for blackbox exporter)

cd ..
cd prometheus
vi prometheus.yml (paste)
change blackbox IP to 13.234.226.74(runnerIP):9115//and modify the URL's you wish to monitor 
http://13.126.49.145:31064 (website//workernodeIP:31064)
http://prometheus.io - 2 targets

we need to restart prometheus
pgrep prometheus (2216)
kill 2216
./prometheus &

goto prometheus dashboard/refresh/status/targets
2 targets are getting monitored in Prometheus (blackbox and Prometheus)
all endpoints are up and running 

for proper visualization: setup graphana
chrome: graphana download - 3 steps to install (at the end to start graphana enter .. 
sudo /bin/systemctl start graphana-server

chrome: pvublicIP:3000 (for Graphana)
default: admin/admin (create new pwd)

goto graphana dashboard/connections/data sources/add data source/select prometheus
provide prometheus url/save and test
right corner/import dashboard
chrome: graphana dashboard blackbox exporter / Prometheus Blackbox Exporter (graphana labs) - ID 7587
copy ID/paste/load
select datasource - prometheus/import

2 targets (http://13.126.49.145:31064 (website) and http://prometheus.io) are getting monitored in Graphana

now setup node exporter:

for website - blackbox exporter
for runner - node exporter

chrome: prometheus.io/download/node exporter for linux
wget (url link)/ls
tar -xvf <tar file> (to unzip)

cd node_exporter
./node_exporter &
enter

chrome: runnerIP:9100

we need to copy below code in Prometheus.yml file again

- job_name: 'node'
  static_configs:
    - targets: ['13.234.226.74:9100']

cd ..
cd prometheus
ls
vi prometheus.yml (add above)
pgrep prometheus
kill 22971
./prometheus &
enter

pometheus dash board: all 3 are up and runing
(node/blackbox/prometheus)

chrome: graphana dashboard node exporter / Prometheus node Exporter (graphana labs) - ID 1860
copy ID/paste/load
select datasource - prometheus/import

click on dashboards
so, we have 2 dashboards 
1. node: cpu, ram etc..
2. blackbox: website and prometheus



