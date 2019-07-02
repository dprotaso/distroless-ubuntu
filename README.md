proof of concept of a distroless ubuntu image

build and export to docker daemon:
```
cd bionic
bazel build //base:all
bazel run //base:debug-nonroot
```

View installed packages
```
docker run -ti bazel/base:debug-nonroot
cd /var/lib/dpkg/status.d
cat *
```
