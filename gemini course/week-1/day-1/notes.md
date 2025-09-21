```shell

#install go
#https://go.dev/dl/go1.25.1.linux-amd64.tar.gz
#sha256:7716a0d940a0f6ae8e1f3b3f4f36299dc53e31b16840dbd171254312c41ca12e

mkdir go-install && cd go-install
curl -OL https://go.dev/dl/go1.25.1.linux-amd64.tar.gz

echo $(echo 7716a0d940a0f6ae8e1f3b3f4f36299dc53e31b16840dbd171254312c41ca12e go1.25.1.linux-amd64.tar.gz) | sha256sum --check
#go1.25.1.linux-amd64.tar.gz: OK

tar -xzvf go1.25.1.linux-amd64.tar.gz && cd go

if [[ -d /usr/local/go ]] then echo exists; else echo invalid; fi

#export PATH=$PATH:/usr/local/go/bin

```
