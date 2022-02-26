# E-Trade API within Docker

* https://developer.etrade.com/getting-started

* https://developer.etrade.com/getting-started/developer-guides#tab_1

* https://developer.etrade.com/support/downloads


```bash
# === create the container
docker run --rm --name etrade ubuntu:20.04 /bin/sleep inf & sleep 2

# === using the here-doc within a here-doc pattern to modify container
docker exec -i etrade /bin/bash <<'eof1'
  ## configure environment
  cat <<'eof' > ~/.bash_aliases
{
  alias cls='clear'
  alias dir='ls -la'
  alias h='history'
  alias more='less -iX'
  export HISTCONTROL=ignoredups:ignorespace
  export HISTFILESIZE=50000
  export HISTSIZE=50000
  export HISTTIMEFORMAT='%t%F %T%t'
  export PAGER='less -iX '
  export IGNOREEOF=20
  export PS1='\u@\h: \w\n\$ '
}
eof

  ## install packages
  apt-get update &&
  apt-get install -y curl vim tree less python3-pip unzip zip python3-venv elinks


  ## get and install E*Trade python client
  cd /tmp/
  [ -f EtradePythonClient.zip ] ||
  curl -O https://cdn2.etrade.net/1/18122609420.0/aempros/content/dam/etrade/developer-site/en_US/document/downloads/EtradePythonClient.zip
  [ -d EtradePythonClient ] || unzip EtradePythonClient.zip 


  ## install python modules
  cd EtradePythonClient
  pip install -r requirements.txt


  ## patch example script
  cd etrade_python_client
  [ -f etrade_python_client.py.orig ] || {
    { cat <<'eof' > patch ; patch -R -b etrade_python_client.py patch ; }
67,68c67
<     print(f'\nPaste this URL into a browser:\n\n{authorize_url}\n\n')
< #    webbrowser.open(authorize_url)
---
>     webbrowser.open(authorize_url)
eof
    }
eof1

docker container commit -p etrade etrade

docker run --rm -v "${PWD}":/config/ --name etrade etrade /bin/sleep inf & sleep 2

docker exec -i etrade /bin/bash <<'eof1'
  ## set up config.ini file
  cd /tmp/EtradePythonClient/etrade_python_client
  [ -f config.ini.orig ] || cp -a config.ini config.ini.orig
  [ -f /config/config.ini ] && sed -e '/PLEASE/d ; s/^;P //' /config/config.ini > config.ini
  [ -f /config/config.ini ] && sed -e '/PLEASE/d ; s/^;S //' /config/config.ini > config.ini
  [ -f /config/config.ini ] || {
    { cat config.ini ; cat << 'eof' ; } > /config/config.ini

; SANDBOX
;S CONSUMER_KEY = 
;S CONSUMER_SECRET = 


; PROD
;P CONSUMER_KEY = 
;P CONSUMER_SECRET = 
eof
    }

eof1

docker exec -it -w /tmp/EtradePythonClient/etrade_python_client etrade /bin/bash
python3 etrade_python_client.py
```

## Running the interactive example client
```text
python3 etrade_python_client.py

1)	Sandbox Consumer Key
2)	Live Consumer Key
3)	Exit
Please select Consumer Key Type: 2

Please accept agreement and enter text code from browser: 

https://api.etrade.com/oauth/request_token

```

