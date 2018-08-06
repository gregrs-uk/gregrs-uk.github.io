---
layout: post
title: "Using a GPG key for SSH authentication on macOS and Debian"
date: 2018-08-06
---

If you have a [GPG](https://gnupg.org/) key, it makes sense to also use it for [SSH](https://en.wikipedia.org/wiki/Secure_Shell) authentication rather than generating a separate key. Since [GnuPG 2.1](https://gnupg.org/faq/whats-new-in-2.1.html) this has become much easier, and whilst there are some good tutorials out there, some are out of date. The basic idea is that instead of using `ssh-agent` for SSH authentication, we'll use `gpg-agent`. I mainly used [bootc's wiki page](https://wiki.bootc.net/gnupg/agent-config-linux.html) and the [notes on incenp.org](https://incenp.org/notes/2015/gnupg-for-ssh-authentication.html), changing a few things in search of a cross-platform solution for macOS 10.12 and Debian 9 so that I have a unified set of config files that can be synced using `git`.

If you don't already have a GPG key/subkey with the `[A]` (authenticate) capability, you'll need to generate one first. I won't describe this process as there are plenty of blog posts out there that do, but in brief I would recommend creating a non-expiring master key with only the `[C]` (certify) capability – perhaps keeping this offline – and expiring subkeys for each other capability, as described in [this post](https://blog.eleven-labs.com/en/openpgp-almost-perfect-key-pair-part-1/). Note however that since [GnuPG 2.1](https://gnupg.org/faq/whats-new-in-2.1.html), you can delete the private part of your master key by deleting the appropriate file (named by keygrip, which you can obtain using `gpg -K --with-keygrip`) in `~/.gnupg/private-keys-v1.d` so you shouldn't need to `--export-secret-subkeys` and re-import them.

Once you have your GPG key, the output from `gpg -K` should look something like the following. Note the `[A]` showing that one of our subkeys has the authenticate capability. It's this subkey that we'll use for SSH authentication.

```
sec   rsa2048 2018-08-05 [C]
      52FEC9F2D4FE99296D1EF4BF1EB7772A98CC62BD
uid           [ultimate] Anne Example <anne@example.org>
ssb   rsa2048 2018-08-05 [S]
ssb   rsa2048 2018-08-05 [E]
ssb   rsa2048 2018-08-05 [A]
```

First we need to add the following to `~/.gnupg/gpg-agent.conf` to enable SSH support in `gpg-agent`.

```
enable-ssh-support
```

Next we'll add the keygrip of the authentication subkey to the `~/.gnupg/sshcontrol` file. This tells `gpg-agent` that we want to use this particular key for SSH authentication. We can find the keygrip using `gpg -K --with-keygrip` and looking for the keygrip associated with the authentication subkey marked `[A]`.

```
sec   rsa2048 2018-08-05 [C]
      52FEC9F2D4FE99296D1EF4BF1EB7772A98CC62BD
      Keygrip = E11D971C7466E818F396867353421EB79BEBF028
uid           [ultimate] Anne Example <anne@example.org>
ssb   rsa2048 2018-08-05 [S]
      Keygrip = 543BEDBDF3D949044EF0C48181BE7C615715F00E
ssb   rsa2048 2018-08-05 [E]
      Keygrip = 80B66EE412ADFE27371A03A1CB2013574707A8F8
ssb   rsa2048 2018-08-05 [A]
      Keygrip = 8FFC951FB9BA2A55A1E26C917DC069EC166D7474
```

In this case the keygrip is `8FFC951FB9BA2A55A1E26C917DC069EC166D7474`, which we add to the file `~/.gnupg/sshcontrol`.

Next we need to add a few commands to our shell startup script. If you use `bash`, add the following into `~/.bash_profile`. I use `zsh` with [oh my zsh](https://ohmyz.sh), so I added the following to a script in `~/.oh-my-zsh/custom/`. These ensure that we are using `gpg-agent`'s socket rather than `ssh-agent`'s and that `gpg-agent` runs when your shell starts.

```
export GPG_TTY=$(tty)
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
gpgconf --launch gpg-agent
```

At this point it's a good idea to restart your shell and run `ssh-add -l`. If all is well you should see your key listed, for example:

```
2048 SHA256:0AJnONdvBRmCIvsIvNsL+/of4uW40NJHZsGytyLDPGQ (none) (RSA)
```

Now we can export the public key in OpenSSH format. Running `gpg --export-ssh-key anne@example.org` (replacing `anne@example.org` with the email address associated with your key) gives the following output, which you should add to `~/.ssh/authorized_keys` on the server to which you're connecting.

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDiIPt7la51CBKTkUgYoq307LgXR6FFsLLaveaiYOtszaf8+ilAT31ywHh8rYOFHtNylTeGCzBT8DWOw1ScwCfuQC/2RtL3WcnNFANnarju9DvA9cqEKkwsZiB92elwnfx3f343lDW7hzvTkEuNAFyPOF4BEWG39TUCcAPCRBeiBa31DPJHfT3l2juLblvH2kOa+bLRGK8ppzb8bWS2O6fKn+QQq5EJpD030YutKsIA0Hq7AIiD72xSJGpttVEXWpNGJHGIpUmy8toWw8QrmXMzqx6lUpwrir+mmXpeMn0nbxOfbxxjmYbX9rhDrnXXegVNmMqVN+dCneVTFGsquw5r openpgp:0x7B4D1004
```

If you have any questions or suggestions, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
