# TCPDUMP

#### Keys

To create a dump file  for wireshark key `-w`

```sh
-w mycap.pcap
```

#### Capture request headers

```sh
sudo tcpdump -i lo0 -vvvs 1024 -l -A dst port 8080
```



