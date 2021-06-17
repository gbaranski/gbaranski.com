+++
title = "Projects"
+++

## Houseflow 

Home automation platform, the project in which I put the most effort. Started it at the very beginning of the programming journey. This project involves a lot of different topics, like networking, electronics, mobile and web development, databases, devops, etc.

Made using 
- [Rust](https://www.rust-lang.org/) for high performance networking services with low memory footprint. My plan is to run that on cheap Raspberry Pi.
- [C++](https://en.m.wikipedia.org/wiki/C%2B%2B) for embedded devices, like ESP8266 or ESP32.
- [PostgreSQL](https://www.postgresql.org/) database for storing users and devices.
- [Redis](https://github.com/redis/redis) database for storing refresh tokens, this allows token invalidation.
- [Websockets](https://en.m.wikipedia.org/wiki/WebSocket) transport protocol for communication between backend and embedded devices.

Previously also
- [Go](https://golang.org/) for OAuth2 Server implementation for Google Smart Home Actions. Resigned because it turned out to be too simple languague, and things like enums, especially for parsing JSON is not possible.	
- [Typescript](https://www.typescriptlang.org/) for Web application and backend
- [React](https://reactjs.org/) for Web application
- [Flutter](https://flutter.dev/) for mobile application.
- [MongoDB](https://www.mongodb.com/) as primary database, abandoned because I wanted to learn more about relational databases.
- [MQTT](https://mqtt.org/) as transport layer protocol between embedded devices and services.

[github.com/gbaranski/houseflow](https://github.com/gbaranski/houseflow)

## LightMQ

Lightweight client-server messaging protocol. Intented to work for [Houseflow](#houseflow), as a replacement to MQTT which don't really fit into my use-case. Now work is continued under new name Lighthouse at [Houseflow](#houseflow).

Made using [Go](https://golang.org/).

[github.com/gbaranski/lightmq](https://github.com/gbaranski/lightmq)

## Cryptogram

Decentralized P2P messaging app. Allows real-time messaging without server.

Made using [Go](https://golang.org/) and [LibP2P](https://libp2p.io/).

[github.com/gbaranski/cryptogram](https://github.com/gbaranski/cryptogram)

## OpenFlavour

My first steps in databases, scraping data from e-cig flavours manufacturers and providing service for [VapeTool](https://vapetool.app/).

Made using Typescript.

[github.com/gbaranski/OpenFlavour-API](https://github.com/gbaranski/OpenFlavour-API) and [github.com/gbaranski/OpenFlavour-Scraper](https://github.com/gbaranski/OpenFlavour-Scraper)

