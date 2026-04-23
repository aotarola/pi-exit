# @aotarola/pi-exit

A tiny pi package that adds `/exit` as an alias for `/quit`.

## Install

### From npm

```bash
pi install npm:@aotarola/pi-exit
```

### From git

```bash
pi install git:https://github.com/aotarola/pi-exit
```

## Try without installing

### From npm

```bash
pi -e npm:@aotarola/pi-exit
```

### From git

```bash
pi -e git:https://github.com/aotarola/pi-exit
```

### From a local checkout

```bash
pi -e .
```

## Usage

After the package is loaded, use either command:

```text
/quit
/exit
```

## What it does

This package registers a single pi slash command:

- `/exit` — graceful alias for `/quit` via `ctx.shutdown()`
