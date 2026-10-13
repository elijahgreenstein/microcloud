(tutorial-ui)=

# Access the UI

Instead of managing your instances and your LXD setup from the command line, you can also use the LXD UI.
See {ref}`lxd:access-ui` for more information.

1. Check the LXD cluster list to determine the IP addresses of the cluster members:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc cluster list

   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   |  NAME  |            URL           |      ROLES       | ARCHITECTURE | FAILURE DOMAIN | DESCRIPTION | STATE  |      MESSAGE      |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro1 | https://10.1.123.10:8443 | database-leader  | x86_64       | default        |             | ONLINE | Fully operational |
   |        |                          | database         |              |                |             |        |                   |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro2 | https://10.1.123.20:8443 | database         | x86_64       | default        |             | ONLINE | Fully operational |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro3 | https://10.1.123.30:8443 | database         | x86_64       | default        |             | ONLINE | Fully operational |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro4 | https://10.1.123.40:8443 | database-standby | x86_64       | default        |             | ONLINE | Fully operational |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   ```

1. In your web browser, navigate to the URL of one of the cluster members.
   For example, for `micro1`, navigate to `https://10.1.123.10:8443`.

1. By default, MicroCloud uses a self-signed certificate, which will cause a security warning in your browser.
   Use your browser’s mechanism to continue despite the security warning.

   ```{figure} /images/ui_security_warning.png
   :alt: Example for a security warning in Chrome

   Example for a security warning in Chrome
   ```

1. You should now see the LXD UI, prompting you to set up a certificate.

   ```{figure} /images/ui_certificate_selection.png
   :alt: Certificate selection in the LXD UI

   Certificate selection in the LXD UI
   ```

   ```{note}
   Since LXD 5.21, the LXD UI is enabled by default.

   If you don't see the certificate screen, you might have an earlier version of LXD. Run `snap info lxd` to check.
   If you have an earlier version, run the following commands on the cluster member that you're trying to access (for example, `micro1`) to enable the UI:

       snap set lxd ui.enable=true
       systemctl reload snap.lxd.daemon
   ```

1. Follow the instructions in the UI to set up the certificates.

    ```{important}
    If you are using virtual machines to complete the tutorial, then you must transfer any new certificates to one of the cluster members to add them to the trust store.

    To do this, use the {ref}`file push command <lxd:instances-access-files-push>`.
    For example:

    ```bash
    lxc file push lxd-ui.crt micro1/root/lxd-ui.crt
    ```

    You can then access the shell on that cluster member and add the certificate to the trust store:

    ```bash
    lxc config trust add lxd-ui.crt
    ```

## Next steps

Now you can use either the CLI or the UI to manage your MicroCloud.
In the next part of the tutorial, you can start {ref}`launching and managing instances <tutorial-instances>`.
