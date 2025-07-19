# dune-combat-app

## Deployment
There is a Github Actions workflow defined in this repo that kicks off on pull requests and pushes to `main` that publishes a Docker image to GHCR at ghcr.io/tdenmon/dune-combat-app.

The app is running on Scaleway's Serverless platform. It requires manual deployments when new changes are pushed up, because I'm too lazy to automate that.

The app is available at https://dune.zeroethangel.space. The DNS is configured manually on DigitalOcean under my `zeroethangel.space` domain.

## Docker Setup
```
docker build . -t dune-combat-app
docker run -it -p 8080:8080 --rm dune-combat-app
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
