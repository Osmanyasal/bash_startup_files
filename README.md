# Bash Startup Files Loading Order

![bash_startup-4](https://user-images.githubusercontent.com/22853419/153561337-dba7aa07-916d-408f-9ab6-5b45f2fed487.png)

Bash reads different configuration files depending on what mode it's lanunched
in. This table lists the possible mode:config-file combinations. The letters in
each cell represent each files respective loading order for that column.

Interacitve logins have a special quirk, only one of the ``*profile`` files is used.
**Between B1, B2, and B3, only the first existing file is sourced. The remaining
config files are ignored.**
<pre>
+--------------------+---------------+---------------+----------+----------+
|  Conf file name    |  Interactive  |  Interactive  |  Script  |  POSIX   |
|                    |  login        |  non-login    |          |  script  |
+====================+===============+===============+==========+==========+
|  /etc/profile      |     A         |               |          |          |
+--------------------+---------------+---------------+----------+----------+
|  /etc/bash.bashrc  |               |      A        |          |          |
|  or /etc/bashrc    |               |               |          |          |
+--------------------+---------------+---------------+----------+----------+
|  ~/.bashrc         |               |      B        |          |          |
+--------------------+---------------+---------------+----------+----------+
|  ~/.bash_profile   |     B1        |               |          |          |
+--------------------+---------------+---------------+----------+----------+
|  ~/.bash_login     |     B2        |               |          |          |
+--------------------+---------------+---------------+----------+----------+
|  ~/.profile        |     B3        |               |          |          |
+--------------------+---------------+---------------+----------+----------+
|  $BASH_ENV         |               |               |    A     |          |
+--------------------+---------------+---------------+----------+----------+
|  $ENV              |               |               |          |    A     |
+--------------------+---------------+---------------+----------+----------+
|  ~/.bash_logout    |      C        |               |          |          |
+--------------------+---------------+---------------+----------+----------+
</pre>

Since each of these config files are actually scripts, it's possible to souce 
one from another. On many distros, it's common to have ``/etc/profile`` source
``/etc/bashrc`` in turn.

If you are concerned that a config file may source another, the only way to know
is to read all the existing config files for the session type yourself (everything
in the relevant column).



Where to Set Environment Variables
----------------------------------
* In most cases, put environment variables in ``/etc/profile`` or ``~/.profile``. Most
  applications in a user session will inherit variables in these files.
* PAM  has facilities for setting environment variables on login, using the
  ``pam_env`` module. You can edit ``/etc/environment``, and if ``pam_env`` is configured
  to read it in ``/etc/security/pam_env.conf``, users will inherit those env vars.
* With systemd units you can specify an ``Environment=`` directive under the ``[Service]`` section.
  More `here <https://coreos.com/os/docs/latest/using-environment-variables-in-systemd-units.html>`_.
* A per-user service manager is started as system service instance of ``user@.service`` for each logged in user.
* There is a more standardized way. Just put your environment files in ``~/.config/environment.d/*.conf``.
  You can read more about this with ``man 5 environment.d``. This also pulls in variables from PAMs
  ``/etc/environment`` file.
* For automation that uses ssh key authenticaion, you can set environment variables
  per-key in ``authorized_keys``. Here's an example, to get a feel for the syntax:
  ``restrict,command="/usr/local/script.backup.sh",environment="automated=1" ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAA...``.
