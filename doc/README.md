- Créer 5 VM avec VirtualBox :
    - debian_redis
    - debian_postgres
    - debian_poll
    - debian_result
    - debian_worker

- Chacune aura 1 Go de RAM ainsi que 10 Go d'espace disque dur au minimum

- Configurer la carte réseau de chaque VM sur `Bridged Adapter`

- Installer Debian 10 Buster (v. debian-10.4.0-amd64-netinst) sans interface graphique sur toutes les VM
    `https://www.howtoforge.com/tutorial/debian-minimal-server/`

- Installer sur chaque VM le serveur SSH, un éditeur shell ainsi que les outils réseaux
        ```sh
        $ su
        $ apt-get -y install ssh openssh-server
        $ apt-get -y install vim-nox nano
        $ apt-get -y install net-tools
        ```

- Redémarrer chaque VM
        ```sh
        $ systemctl reboot
        ```

- Vérifier sur chaque VM que le fichier `/etc.apt/sources.list` contient le repository `buster/updates` et activer les repositories `contrib` et `non-free`
        ```sh
        $ nano /etc/apt/sources.list

            deb http://deb.debian.org/debian/ buster main contrib non-free
            deb-src http://deb.debian.org/debian/ buster main contrib non-free

            deb http://security.debian.org/debian-security buster/updates main contrib non-free
            deb-src http://security.debian.org/debian-security buster/updates main contrib non-free
        ```

- Mettre à jour la base de données du package `apt` et installer les dernières mises à jour sur chaque VM
        ```sh
        $ apt-get update
        $ apt-get upgrade
        ```

- Afficher l'IP de chaque VM
        ```sh
        $ ip a
        ```

- Déclarer l'IP de chaque VM dans le fichier `/etc/hosts` du node manager en leur associant un alias (queue1, db1, webapp1, webapp2, backend1)

- Installer les paquets `python` et `sshpass`
        ```sh
        $ sudo apt-get -y install python
        $ sudo apt-get -y install sshpass
        ```

- Installer Ansible sur le node manager
        ```sh
        $ sudo apt-get update
        $ sudo apt-get -y upgrade
        $ sudo apt-add-repository ppa:ansible/ansible
        $ sudo apt-get update
        $ sudo apt-get -y install ansible
        ```

- Créer le fichier inventaire Ansible
        ```sh
        $ cd DP_automation_2019
        $ nano inventory.ini

            [redis]
            queue1

            [postgres]
            db1

            [poll]
            webapp1

            [result]
            webapp2

            [worker]
            backend1
        ```

- Vérifier que le node manager communique avec les nodes et enregistrer la fingerprint
        ```sh
        $ ssh root@queue1
        $ ssh root@db1
        $ ssh root@webapp1
        $ ssh root@webapp2
        $ ssh root@backend1
        ```

- Pinger tous les nodes
        ```sh
        $ ansible -i inventory.ini -m ping all --user root --ask-pass
        ```

- Vérifier que Python est installé sur tous les nodes
        ```sh
        $ ansible -i inventory.ini -m raw -a "apt-get install -y python2" all --user root --ask-pass
        ```

- Générer un mot de passe chiffré
        ```sh
        $ ansible localhost -i inventory.ini -m debug -a "msg={{ 'passforce' | password_hash('sha512', 'secretsalt') }}"
        ```

- Créer l'utilisateur `user-ansible` avec un mot de passe chiffré sur tous les nodes
        ```sh
        $ ansible -i inventory.ini -m user -a 'name=user-ansible password=[mot_de_passe_chiffré]' --user root --ask-pass all
        ```

- Installer la commande `sudo` sur tous les nodes
        ```sh
        $ ansible -i inventory.ini -m raw -a "apt-get install sudo" all --user root --ask-pass
        ```

- Ajouter l'utilisateur `user-ansible` au groupe `sudo`
        ```sh
        $ ansible -i inventory.ini -m user -a 'name=user-ansible groups=sudo append=yes' --user root --ask-pass all
        ```

- Vérifier que l'utilisateur a bien les droits `sudo` (mot de passe : `passforce`)
        ```sh
        $ ansible -i inventory.ini -m user -a 'name=user-ansible groups=sudo append=yes' --user user-ansible --ask-pass --become --ask-become-pass all
        ```

- Créer une paire de clés SSH de type ECDSA
        ```sh
        $ ssh-keygen -t ecdsa
        ```

- Ajouter la clé publique de l’utilisateur `user-ansible` sur tous les nodes
        ```sh
        $ ansible -i inventory.ini -m authorized_key -a 'user=user-ansible state=present key="{{ lookup("file", "/home/[node_manager]/.ssh/id_ecdsa.pub") }}"' --user user-ansible --ask-pass --become --ask-become-pass all
        ```

- Relancer la commande sans le drapeau `--ask-pass` pour vérifier que les commandes distantes peuvent être exécutées avec Ansible avec le privilège `sudo`
        ```sh
        $ ansible -i inventory.ini -m authorized_key -a 'user=user-ansible state=present key="{{ lookup("file", "/home/[node_manager]/.ssh/id_ecdsa.pub") }}"' --user user-ansible --become --ask-become-pass all
        ```

- Chiffrer des chaînes de caractères avec la commande `ansible-vault`
        ```sh
        $ ansible-vault encrypt_string 'foobar' --name '[nom_de_variable]'
        ```

- Lancer la commande `ansible-playbook` pour exécuter le playbook
        ```sh
        $ ansible-playbook -i inventory.ini --user user-ansible --become --ask-become-pass playbook.yml
        ```
