# install @vue/cli-service

There is something strange in the neighborhood...

## dev-environment

Build the development-container:
```
$ docker build --tag flipflop .
```

Run the development-container:
```
$ docker run -it --rm --user $(id -u):$(id -g) --mount type=bind,src="$(pwd)",dst=/app flipflop
```

Inside the dev-container, we have node (v12.18.4) and npm (v6.14.6).
