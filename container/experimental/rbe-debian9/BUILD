# Copyright 2017 The Bazel Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

licenses(["notice"])  # Apache 2.0

package(default_visibility = ["//visibility:public"])

load(
    "//rules/container:docker_toolchains.bzl",
    "language_tool_layer",
    "toolchain_container",
)
load("@io_bazel_rules_docker//contrib:test.bzl", "container_test")
load("//container/common/clang:clang.bzl", "clang_env")

JAVA_CLEANUP_COMMANDS = (
    "rm -rf " +
    "/etc/ssl/certs/java/cacerts " +
    "/tmp/hsperfdata_root/* " +
    "/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/amd64/server/classes.jsa"
)

PYTHON_CLEANUP_COMMANDS = (
    "find /usr/lib/python* -name __pycache__ -exec rm -rf {} +"
)

toolchain_container(
    name = "toolchain",
    base = "@debian9//image",
    cmd = [
        "/bin/sh",
        "-c",
        "/bin/bash",
    ],
    env = {
        # PATH envvar is a special case, and currently only the one in the
        # topmost layer is set. So that we override it here to include all.
        "PATH": "$PATH:/usr/local/go/bin",
        "LANG": "C.UTF-8",
        "LANGUAGE": "C.UTF-8",
        "LC_ALL": "C.UTF-8",
    },
    language_layers = [
        "base-ltl",
        "clang-ltl",
        "go-ltl",
        "java-ltl",
        "java10-ltl",
        "python-ltl",
    ],
)

language_tool_layer(
    name = "base-ltl",
    base = "@debian9//image",
    packages = [
        "binutils",
        "ca-certificates",
        "curl",
        "ed",
        "file",
        "git",
        "less",
        "netcat",
        "openssh-client",
        "patch",
        "unzip",
        "wget",
        "zip",
    ],
)

language_tool_layer(
    name = "clang-ltl",
    base = "@debian9//image",
    env = clang_env,
    packages = [
        "libstdc++-6-dev",
    ],
    tars = [
        "//third_party/clang:debian9_tar",
        "//third_party/libcxx:debian9_tar",
    ],
)

language_tool_layer(
    name = "go-ltl",
    base = "@debian9//image",
    env = {
        "GOPATH": "/go",
        "PATH": "$PATH:/usr/local/go/bin",
    },
    tars = ["//third_party/golang:tar"],
)

language_tool_layer(
    name = "java-ltl",
    additional_repos = [
        "deb http://deb.debian.org/debian stretch-backports main",
    ],
    base = "@debian9//image",
    env = {
        "JAVA_HOME": "/usr/lib/jvm/java-8-openjdk-amd64",
    },
    installation_cleanup_commands = JAVA_CLEANUP_COMMANDS,
    packages = [
        "ca-certificates-java",
        "openjdk-8-jdk-headless",
        "openjdk-8-jre-headless",
    ],
    symlinks = {
        "/usr/bin/java": "/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java",
    },
)

language_tool_layer(
    name = "java10-ltl",
    base = "@debian9//image",
    tars = [
        "//third_party/openjdk:open_jdk_tar",
        "//third_party/openjdk:open_jdk_src_tar",
    ],
)

language_tool_layer(
    name = "python-ltl",
    additional_repos = [
        "deb http://httpredir.debian.org/debian testing main",
    ],
    base = "@debian9//image",
    installation_cleanup_commands = PYTHON_CLEANUP_COMMANDS,
    packages = [
        "python-dev",
        "python-numpy",
        "python-pip",
        "python-setuptools",
        "python3.6",
        "python3.6-dev",
        "python3-numpy",
        "python3-pip",
        "python3-setuptools",
    ],
    symlinks = {
        "/usr/bin/python": "/usr/bin/python2.7",
    },
)

container_test(
    name = "toolchain-test",
    configs = [
        ":rbe-debian9.yaml",
        "//container/common:go.yaml",
        "//container/common:java.yaml",
        "//container/common:python2.yaml",
        "//container/common:rbe-base.yaml",
        "//container/common/clang:clang.yaml",
    ],
    image = ":toolchain",
    verbose = True,
)
