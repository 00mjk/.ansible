# .ansible

workstation bootstrapping, to be rebased onto a blank kde neon install

- tested with neon 5.12.x
- uses ansible 2.5.x via PPA so human-readable output via callback works
- default verbosity changed to '-vv' so the FILE:LINE position of the currently run task is shown

## bootstrapping

### make ssh work, so ansible behaves
    sudo bash -c "echo 'sjas ALL=(ALL:ALL) NOPASSWD:ALL' > /etc/sudoers.d/sjas
    sudo apt install openssh-server -y
    ssh-keygen -trsa -b4096 -N '' -f ~/.ssh/id_rsa
    ssh-copy-id localhost  ## enter password here when promted
    mkdir -p ~/.ssh/controlmasters

### setup ansible itself
    sudo apt install software-properties-common -y
    sudo apt-add-repository ppa:ansible/ansible -y
    sudo apt update
    sudo apt install ansible -y
    ANSIBLETEMPROOT=~/etc
    mkdir -p $ANSIBLETEMPROOT
    cd $ANSIBLETEMPROOT
    git clone https://github.com/sjas/.ansible
    cat << EOF > ~/.ansible.cfg
    [defaults]
    inventory = $ANSIBLETEMPROOT/.ansible/hosts
    stdout_callback = debug
    verbosity = 2
    [ssh_connection]
    ssh_args = -o controlmaster=auto -o controlpersist=60s -o controlpath=~/.ssh/controlmasters/%r@%h:%p
    pipelining = yes
    [diff]
    always = yes
    context = 4
    EOF
    cd .ansible

## run ansible to get the workstation up and running
    cd ~/etc/.ansible
    ansible-playbook neon.yml

## post tasks

    # make your new settings known to your current shell
    . ~/.bashrc
    # logout/login to KDE
    # otherwise your GUI changes will not be applied

## some useful oneliners for discerning settings' locations

    find . -type f -ls | grep .git -v > asdf1; read; find . -type f -ls | grep .git -v > asdf2; diff asdf{1,2} | awk 'NF>1 {print $NF}' | sort -u | awk 'NF>0'| grep -ve 'asdf[12]' | sed -e 's/^\.\///' ; rm asdf{1,2}

    strace systemsettings5 |& grep ^stat | grep -v -e ENOENT -e /etc/localtime -e st_mode=S_IFDIR -e /run/user/1000 -e /usr/lib/x86_64- | cut -c7- | sed -e 's/"/ /' | awk '{print $1}'

    watch -n1 -d 'git diff HEAD | grep -e diff -e \+\+\+ -e^\+ | grep --color -e$ -ediff\ \-\-.\* | wc -l; echo; git status'