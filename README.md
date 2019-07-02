### proof of concept of a distroless ubuntu image

1. build and export to docker daemon:
```
cd bionic
bazel build //base:all
bazel run //base:debug-nonroot
```

2. view installed packages
```
docker run -ti bazel/base:debug-nonroot
cd /var/lib/dpkg/status.d
cat *
```
