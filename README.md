# Nginx multi vhost SSL redirect

# local testing 

add following content to /etc/hosts. This is needed to make redirection succeed
```
127.0.1.1       site1.lan
127.0.1.1       site2.lan
127.0.1.1       site3.lan
```

# SSL

Used following oneliner for certificate and key
```
echo {1..3} | tr ' ' '\n' | while read id; do openssl req -nodes -x509 -keyout server${id}.key -out server${id}.crt -sha256 -days 365 -subj "/C=US/ST=Oregon/L=Portland/O=Company Name/OU=Org/CN=site${id}lan"; done
```

# run container

Just execute 
```
docker-compose up
```

# curl

And watch output
```
curl -H "Host: site1.lan" localhost:80 -k -v  -L 2>&1 | grep -E '(Host)|(https)|(Site)'
curl -H "Host: www.site1.lan" localhost:80 -k -v  -L 2>&1 | grep -E '(Host)|(https)|(Site)'
```

output:
```
# request to
> Host: site1.lan

# answer with redirect
< Location: https://site1.lan/

# execute redirect
* Issue another request to this URL: 'https://site1.lan/'
> Host: site1.lan

# profit
<title>Welcome to Site 1!</title>
<h1>Welcome to Site 1!</h1>

# Same as above, but with www. subdomain
> Host: www.site1.lan
< Location: https://site1.lan/
* Issue another request to this URL: 'https://site1.lan/'
> Host: site1.lan
<title>Welcome to Site 1!</title>
<h1>Welcome to Site 1!</h1>
```

Use oneliner, to test them all
```
echo {1..3} | tr ' ' '\n' | while read id; do \
echo "==== {*.*} Testing site${id}.lan ===="; \
curl -H "Host: site${id}.lan" localhost:80 -k -v  -L 2>&1 | grep -E '(Host)|(https)|(Site)'; \
echo "==== {*.*} Testing www.site${id}.lan ===="; \
curl -H "Host: www.site${id}.lan" localhost:80 -k -v  -L 2>&1 | grep -E '(Host)|(https)|(Site)'; \
echo; \
done
```

