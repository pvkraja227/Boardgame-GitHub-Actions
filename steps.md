https://www.youtube.com/watch?v=FTrTFOLbdm4&t=1666s
The Ultimate CICD Corporate DevOps Pipeline Project | Real-Time DevOps Project | GitHub Actions

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
automatically creates /.github/workflows (name pipeline of your wish CICD.yml
(modify pipeline)
name: CICD Pipeline
(delete pull, which means any changes to push, pipeline will be triggered)
runs-on: self-hosted
run: mvn package --file pom.xml


