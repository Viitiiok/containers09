# containers09

## Scopul lucrării

Acest raport explică pașii de optimizare a imaginilor Docker și evidențiază metode practice de optimizare a imaginilor.

## Sarcina

Compararea următoarelor metode de optimizare a imaginilor:

Ștergerea fișierelor temporare și a dependențelor neutilizate

Reducerea numărului de straturi

Utilizarea unei imagini de bază minime

Reambalarea imaginii

Combinarea tuturor metodelor

## Pașii de realizare

Clonare și structură proiect
git clone https://github.com/username/containers09.git
cd containers09
mkdir site

Imaginea de bază (raw)

Crează fișierul Dockerfile.raw cu:

FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y nginx
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Construcție:

docker build -t mynginx:raw -f Dockerfile.raw .

Ștergerea fișierelor temporare și a dependențelor neutilizate (clean)

Crează Dockerfile.clean:

FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y nginx
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Construcție și verificare dimensiune:

docker build -t mynginx:clean -f Dockerfile.clean .
docker image list | grep mynginx:clean

Minimizarea numărului de straturi (few)

Crează Dockerfile.few cu toate comenzile RUN într-un singur strat:

FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Construcție:

docker build -t mynginx:few -f Dockerfile.few .
docker image list | grep mynginx:few

Utilizarea unei imagini de bază minime (alpine)

Crează Dockerfile.alpine:

FROM alpine:latest
RUN apk update && apk upgrade && apk add nginx
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Construcție:

docker build -t mynginx:alpine -f Dockerfile.alpine .
docker image list | grep mynginx:alpine

Reambalarea imaginii raw 

docker container create --name temp raw mynginx:raw
docker container export temp | docker image import - mynginx:repack
docker container rm temp
docker image list | grep mynginx:repack

Combinarea tuturor metodelor (minx + repack)

Crează Dockerfile.min folosind Alpine și cleanup:

FROM alpine:latest
RUN apk update && apk upgrade && \
    apk add nginx && rm -rf /var/cache/apk/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Construcție și reambalare:

docker build -t mynginx:minx -f Dockerfile.min .
docker container create --name temp mynginx:minx
docker container export temp | docker image import - mynginx:min
docker container rm temp
docker image list | grep mynginx:minx\|mynginx:min
<img width="1026" alt="Screenshot 2025-04-27 at 23 56 36" src="https://github.com/user-attachments/assets/e40658ae-b221-46bc-abed-99c2d468cfcc" />
<img width="1030" alt="Screenshot 2025-04-27 at 23 55 58" src="https://github.com/user-attachments/assets/dc564066-419c-43c6-ad2f-26b63ef73641" />
<img width="1041" alt="Screenshot 2025-04-27 at 23 55 08" src="https://github.com/user-attachments/assets/0dffe873-9b13-4aa1-994c-a6d9a1798d6d" />
<img width="1792" alt="Screenshot 2025-04-27 at 23 54 42" src="https://github.com/user-attachments/assets/13b824cc-1ecb-4a13-bfd6-cb3e8fde395e" />
<img width="1791" alt="Screenshot 2025-04-27 at 23 53 45" src="https://github.com/user-attachments/assets/4dcf89c5-6d65-4fa6-b32a-591245f48c34" />
<img width="1789" alt="Screenshot 2025-04-27 at 23 53 11" src="https://github.com/user-attachments/assets/b969d5f2-5cba-404a-a319-8d014e31a5d6" />

## Concluzii

Folosirea unei imagini de bază minimă (Alpine) reduce major dimensiunea.

Combinarea comenzilor într-un singur strat ajută.

Reambalarea poate elimina metadata suplimentară.

Metoda finală (minx + reambalare) oferă cel mai mic footprint.

## Întrebări și răspunsuri

Care metodă vi se pare cea mai eficientă?

Combinația Alpine + cleanup + minimizarea stratificării și reambalare.

De ce curățarea cache-ului pachetelor într-un strat separat nu reduce dimensiunea imaginii?

Pentru că stratul anterior rămâne în istoric. Ștergerea cache-ului într-un pas nou nu afectează datele deja stocate în stratul anterior.

Ce este reambalarea imaginii?

Exportarea containerului ca tar și importul ca imagine nouă. Astfel se unifică straturile și se pot elimina resturile de metadata.


