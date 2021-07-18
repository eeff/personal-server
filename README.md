# Personal Server Management

Manage personal servers inspired by [erebe].

## Prerequisites

### Software Requirements

* [ansible] >= 2.10.0
* [GnuPG] >= 2.1.17
* [OpenSSH] >= 6.7

### Ansible dependencies

Run the following the command to install ansible dependencies:

``` shell
ansible-galaxy install -r requirements.yml
```

### Creating GPG keys

Follow [creating-the-perfect-gpg-keypair] to create the keys.
Or better, if you have a [YubiKey] smart card, follow [drduh]'s guide on using
GPG with YubiKey.

### Creating SSH keys

The common way is to generate new SSH keys using `ssh-keygen`, but since
[gpg-agent] has OpenSSH agent emulation, you could just use a PGP key for SSH
authentication. The latter has the benefit of reducing key maintenance and the
ability to store the authentication key on a smartcard.

Once you create the GPG keys in the previous step, there is a subkey with
`Authentication` capability which could be used for SSH authentication.

Enable the gpg-agent ssh support:

``` shell
echo enable-ssh-support >> $GNUPGHOME/gpg-agent.conf
```

Set `SSH_AUTH_SOCK` so that SSH will use `gpg-agent` instead of `ssh-agent`.
Add this to your bashprofile or zshrc

``` shell
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$  ]; then
  export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
  fi
  export GPG_TTY=$(tty)
  gpg-connect-agent updatestartuptty /bye >/dev/null
```

If you are not using [YubiKey], then you need to find the authentication subkey
keygrip, and add the keygrip in the list of approved keys:

``` shell
gpg --list-keys --with-keygrip
echo $authentication_keygrip >> $GNUPGHOME/sshcontrol
```

Check that the key is present in the ssh identities list:

``` shell
ssh-add -L
```

[ansible]: https://www.ansible.com
[creating-the-perfect-gpg-keypair]: https://alexcabal.com/creating-the-perfect-gpg-keypair
[drduh]: https://github.com/drduh/YubiKey-Guide
[erebe]: https://github.com/erebe/personal-server
[gpg-agent]: https://wiki.archlinux.org/title/GnuPG#SSH_agent
[gpg-ssh-setup]: https://gist.github.com/mcattarinussi/834fc4b641ff4572018d0c665e5a94d3
[GnuPG]: https://gnupg.org
[OpenSSH]: https://openssh.com
[YubiKey]: https://www.yubico.com/products/yubikey-hardware/
