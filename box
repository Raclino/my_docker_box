#!/usr/bin/env python3

import sys
import os
import yaml


def chroot(chroot_dir):
    os.chroot(chroot_dir)
    os.chdir("/")


def unchroot(real_root):
    os.fchdir(real_root)
    os.chroot(".")
    os.chdir('/')
    os.close(real_root)


def mount(env_name):
    os.chdir(env_name)
    os.system("mount -t proc /proc proc/ && mount --bind /sys sys/")
    if not os.path.exists('dev/null'):
        os.system("mknod -m 666 dev/null c 1 3")
    if not os.path.exists('dev/random'):
        os.system("mknod -m 666 dev/random c 1 8")
    if not os.path.exists('dev/urandom'):
        os.system("mknod -m 666 dev/urandom c 1 9")
    os.chdir('/')


def umount(env_name):
    os.system('umount ' + env_name + '/sys')
    os.system('umount ' + env_name + '/proc')


def set_env_name(base_env_name):
    print("Setting environment name ...")
    a = 0
    while os.path.exists('/var/lib/box/env/' + base_env_name + str(a)):
        a += 1
    print("Environment name set.")
    return base_env_name + str(a)


def copy_env(env_name):
    print("Duplicating environment...")
    os.system('cp -r /var/lib/box/base ' + env_name)
    print("Environment duplicated")


def base_settings(env_name):
    real_root = os.open("/", os.O_RDONLY)
    chroot(env_name)
    os.system("chmod 1777 /tmp")
    os.system("echo 'nameserver 8.8.8.8' > /etc/resolv.conf")
    os.system("apt-get update && apt-get upgrade -y")
    os.system("apt-get install wget gnupg -y")
    os.system("touch /run/conf.yml")
    os.mkdir('/data')
    os.mkdir('/data/db')
    unchroot(real_root)


def env_settings(env_name, key, repository, requirements, user):
    real_root = os.open("/", os.O_RDONLY)
    chroot(env_name)
    if key is not None:
        os.system('wget -qO - ' + key + ' | apt-key add -')
    if repository is not None:
        os.system('echo ' + repository + ' >> /etc/apt/sources.list')
        os.system('apt-get update')
    if requirements is not None:
        os.system('apt-get install -y ' + requirements)
    if user is not None:
        os.system("useradd " + user + " -s /bin/bash -p '*'")
    unchroot(real_root)


def build(path_to_yml_file):
    if os.path.isfile(path_to_yml_file):
        yaml_file = open(path_to_yml_file, 'r')
        yaml_content = yaml.safe_load(yaml_file)
        repositories = yaml_content.get("repositories")
        repositories = yaml.dump(repositories)
        repositories = yaml.safe_load(repositories)
        requirements = yaml_content.get("requirements")
        key = None
        repository = None
        if repositories is not None:
            key = repositories.get('key')
            repository = repositories.get('repository')
        env_name = set_env_name(yaml_content.get('name'))
        path_to_env = "/var/lib/box/env/" + env_name
        run = yaml_content.get('run')
        user = yaml_content.get('user')
        copy_env(path_to_env)
        mount(path_to_env)
        base_settings(path_to_env)
        env_settings(path_to_env, key, repository, requirements, user)
        yaml_cp = open(path_to_env + '/run/conf.yml', '+w')
        yaml.dump(yaml_content, yaml_cp)
        umount(path_to_env)
        print("Environment created and named: " + env_name)
    else:
        print("Can't read this file")


def cmd_def(cmd_array, start):
    cmd = ""
    if "$ARGS" in cmd_array:
        index = cmd_array.index('$ARGS')
        for word in cmd_array[:index]:
            cmd += word
            cmd += " "
        for arg in sys.argv[start:]:
            cmd += arg
            cmd += " "
    else:
        for word in cmd_array:
            cmd += word
            cmd += " "
    cmd = cmd[:-1]
    return cmd


def run():
    curent_dir = os.getcwd()
    if sys.argv[2] == "--share":
        env_name = sys.argv[4]
        start = 5
        files = sys.argv[3].split(':')
    else:
        env_name = sys.argv[2]
        start = 3
        files = None
    path_to_env = "/var/lib/box/env/" + env_name
    if os.path.isdir(path_to_env):
        yaml_file = open(path_to_env + '/run/conf.yml', 'r')
        yaml_content = yaml.safe_load(yaml_file)
        user = yaml_content.get('user')
        cmd = cmd_def(yaml_content.get('run').split(), start)
        mount(path_to_env)
        if files is not None:
            parent_dir = os.path.dirname(path_to_env + files[1])
            if os.path.isdir(parent_dir):
                os.chdir(curent_dir)
                os.system("cp " + files[0] + " " + path_to_env + files[1])
            else:
                print("'" + parent_dir + "' does not exit")
                files = None

        real_root = os.open("/", os.O_RDONLY)
        chroot(path_to_env)
        if user is not None:
            os.system("chown " + user + ' ' + files[1] )
            cmd = "runuser -u " + user + " -- " + cmd
        os.system(cmd)
        unchroot(real_root)
        umount(path_to_env)
        os.chdir(curent_dir)
        if files is not None:
            os.system('pwd')
            os.system("cp " + path_to_env + files[1] + " " + files[0])
            os.system("rm " + path_to_env + files[1])

    else:
        print(env_name)
        print("Environment '" + env_name + "' does not exist")


def main():
    if len(sys.argv) < 2:
        print("Missing arguments")
        exit(1)
    if str(sys.argv[1]) == "build":
        if len(sys.argv) < 3:
            print("You need to set a path to a .yml file")
        else:
            print(str(sys.argv[2]))
            build('./' + str(sys.argv[2]))
    elif str(sys.argv[1]) == "run":
        run()
    elif str(sys.argv[1]) == "list":
        for env in os.listdir("/var/lib/box/env"):
            print(env)
    else:
        print("Unknown argument")
    exit(0)


if __name__ == "__main__":
    main()
