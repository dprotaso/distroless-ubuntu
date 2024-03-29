package(default_visibility = ["//visibility:public"])

load("@bazel_tools//tools/build_defs/pkg:pkg.bzl", "pkg_tar")
load("@io_bazel_rules_docker//container:container.bzl", "container_image")
load("@io_bazel_rules_docker//contrib:test.bzl", "container_test")
load("@io_bazel_rules_docker//contrib:passwd.bzl", "passwd_entry", "passwd_tar")
load("@io_bazel_rules_docker//contrib:group.bzl", "group_entry", "group_file")

load("@distroless//cacerts:cacerts.bzl", "cacerts")
load("@package_bundle//file:packages.bzl", "packages")

# Create /etc/passwd with the root user
passwd_entry(
    name = "root_user",
    gid = 0,
    home = "/root",
    info = "root",
    shell = "/sbin/nologin",
    uid = 0,
    username = "root",
)

NOBODY = 65534

NONROOT = 65532

passwd_entry(
    name = "nobody_user",
    create_home = False,
    gid = NOBODY,
    home = "/nonexistent",
    info = "nobody",
    shell = "/sbin/nologin",
    uid = NOBODY,
    username = "nobody",
)

passwd_entry(
    name = "nonroot_user",
    gid = NONROOT,
    home = "/home/nonroot",
    info = "nonroot",
    shell = "/sbin/nologin",
    uid = NONROOT,
    username = "nonroot",
)

passwd_tar(
    name = "passwd",
    entries = [
        ":root_user",
        ":nobody_user",
        ":nonroot_user",
    ],
    passwd_file_pkg_dir = "etc",
)

# Create /etc/group with the root, tty, and staff groups
group_entry(
    name = "root_group",
    gid = 0,
    groupname = "root",
)

group_entry(
    name = "nobody_group",
    gid = NOBODY,
    groupname = "nobody",
)

group_entry(
    name = "nonroot_group",
    gid = NONROOT,
    groupname = "nonroot",
)

group_entry(
    name = "tty_group",
    gid = 5,
    groupname = "tty",
)

group_entry(
    name = "staff_group",
    gid = 50,
    groupname = "staff",
)

group_file(
    name = "group",
    entries = [
        ":root_group",
        ":nobody_group",
        ":tty_group",
        ":staff_group",
        ":nonroot_group",
    ],
)

pkg_tar(
    name = "group_tar",
    srcs = [":group"],
    mode = "0644",
    package_dir = "etc",
)


cacerts(
    name = "cacerts",
    deb = packages["ca-certificates"],
)

# Create /tmp, too many things assume it exists.
# tmp.tar has a /tmp with the correct permissions 01777
# A tar is needed because at the moment there is no way to create a
# directory with specific permissions.

container_image(
    name = "static",
    debs = [
        packages["base-files"],
        packages["netbase"],
        packages["tzdata"],
    ],
    env = {
        "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        # allows openssl to find the certificates by default
        # TODO: We should run update-ca-certifaces, but that requires "openssl rehash"
        # which would probably need to be run inside the container
        "SSL_CERT_FILE": "/etc/ssl/certs/ca-certificates.crt",
    },
    tars = [
        ":passwd",
        ":group_tar",
        ":tmp.tar",
        ":nsswitch.tar",
        "@bionic//file:os_release.tar",
        ":cacerts.tar",
    ],
)

container_image(
    name = "base",
    base = ":static",
    debs = [
        packages["libc6"],
        packages["libssl1.1"],
        packages["openssl"],
    ],
)

# A debug image with busybox available.
container_image(
    name = "debug",
    base = ":base",
    directory = "/",
    entrypoint = ["/busybox/sh"],
    env = {"PATH": "$$PATH:/busybox"},
    tars = ["//busybox:busybox.tar"],
)

# Non-root base images
container_image(
    name = "static-nonroot",
    base = ":static",
    user = "%d" % NONROOT,
    workdir = "/home/nonroot",
)

container_image(
    name = "base-nonroot",
    base = ":base",
    user = "%d" % NONROOT,
    workdir = "/home/nonroot",
)

container_image(
    name = "debug-nonroot",
    base = ":debug",
    user = "%d" % NONROOT,
    workdir = "/home/nonroot",
)

container_test(
    name = "debug_test",
    configs = ["testdata/debug.yaml"],
    image = ":debug",
)

container_test(
    name = "base_test",
    configs = ["testdata/base.yaml"],
    image = ":base",
)

