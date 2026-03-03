# Grafana

## Install Grafana on Docker
```
docker run -d --name=grafana -p 3000:3000 -v grafana-data:/var/lib/grafana -v grafana-conf:/usr/share/grafana grafana/grafana
```

Access grafana dashboard
```
http://172.23.x.x:3000
```

Login 
```
username: admin
password: admin
```
