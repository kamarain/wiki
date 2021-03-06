# How to Set up Remote Access

_This is the guide for setting up the remote connection to your office machine and it will not tell you how to set up the machine itself (e.g. install Ubuntu, allocate disk space). If you would like some guidance on this topic, for example, you would want to make your remote machine to "look" like the rest of our machines, please refer to [this unofficial guide](https://github.com/v-iashin/TuniSurvivalKit/blob/master/how_to_setup_a_desktop.md) – also, let us know if it was useful and whether the wiki might benefit from having it here._

We assume that you have an Ubuntu 16.04 or 18.04 desktop (**host**) at your office and you would like to access it remotely from, let's say, your personal laptop (**client**). If the machine is already set up and you would like to just learn how to connect to it, use only the guide for a client.

The connection will be maintained using a proxy server (`ssh-forward.tuni.fi`). Currently, this is the most convenient and stable way of setting up the remote connection to your compute machine (to the best of our knowledge – let us know if there is something else).

The host machine will be reachable using a University-maintained laptop using University `TUNI-STAFF` WiFi or a pre-installed TUNI VPN, for a self-maintained laptop you will need to do something extra and we have it in this guide.

## Known Issues

- You will need to contact `it-helpdesk@tuni.fi` and ask to activate and configure your wall internet socket in a special way. This might take some time.
- `ssh-forward.tuni.fi` doesn't support key-pair authentication.
- Depending on the network you are using, a different log in procedure will be required: only your TUNI password if you are connected to `roam.fi/eduroam/TUNI-STAFF`; and 2FA + your TUNI and host-machine passwords  on other networks.
- `ssh-forward.tuni.fi` has very limited disk space for each user (few MB). Therefore, can only be used as a proxy for your `ssh` connection *which is its main purpose*.

## How to Set up the **Host** (e.g. compute machine)?
1. Email `it-helpdesk@tuni.fi` to connect your office machine to `pit.cs.tut.fi` network. They will assign a fixed IP/FQDN and you will not need to type your credentials every 24 hours to have an internet connection. Specify the following things:
    - the inventory number of the machine (on the sticker),
    - MAC address of the socket in the machine you would like to use for the wired connection to the internet (you may have several Ethernet ports – you need only one),
    - mention the Ethernet socket number from the wall that you will use.
2. At this point, you should have had received the response from `it-helpdesk@tuni.fi` and be able to connect to the internet using the socket you specified. If so, check your IP and type `host your_IP` to find out the FQDN. It should be something like `IP_reversed pointed to **********.pit.cs.tut.fi`.
3. Install `openssh-server` on your machine (host). This will allow `ssh` connection to this machine.
4. Next, make sure **no** WiFi connection connects automatically after the startup.  Type `sudo nm-connection-editor` in terminal (or just go to `Edit connection` from the status menu on 16.04).
5. Allow your `Wired connection` to automatically connect when available.

## How to Set up the **Client** (e.g. your laptop)?
If you have a machine which is already connected to `pit.cs.tut.fi`, you will only need to follow this guide.

1. Now your machine (host) should be reachable from TUNI-maintained computers connected to `TUNI-STAFF` WiFi or VPN directly via `ssh`. To connect to it from a self-maintained/personal device, you need to get access to `ssh-forward.tuni.fi`. For this, proceed to [id.tuni.fi/idm](https://id.tuni.fi/idm/?uiLang=en) -> `My user rights` -> `Apply for a new user right` -> if required, select your contract -> search for `Linux Servers (LINUX-SERVERS) SSH tunneling service` and select it. In `Application details` put something like `For pit.cs.tut.fi connections`. Then, go to the `Applications` tab and wait until the access is granted (1 min).
2. Initialize the two-step verification at `ssh-forward.tuni.fi`. For this, `ssh` with your TUNI credentials to `ssh-forward.tuni.fi` while being connected to one of the University networks (`roam.fi/eduroam/TUNI-STAFF` or a university VPN) – you cannot do it from any other network but [we got you here as well](#how-to-initialize-the-two-step-verification-remotely). Type `google-authenticator`. It will ask you several questions and show a QR code (resize your window to see it). Answer the questions as follows:
    - `Do you want authentication tokens to be time-based` -> y
    - `Do you want me to update your "/home/user/...google_authenticator" file` -> y
    - `Do you want to disallow multiple uses...` -> n
    - `By default, tokens are good for 30 seconds` -> n
    - `If the computer that you are logging into` -> y
3. Once QR code is shown, install some two-step authenticator app on your smartphone e.g. from [Microsoft](https://www.microsoft.com/en-us/account/authenticator), [Authy](https://authy.com/), or [Google](https://www.google.com/search?q=Google+Authenticator+apple+android). The app will be used for two-step authentication when you connect from a non-university network is used e.g. your home internet. Once it is done, it will create an entry with a 6-digit passcode which changes every 30 secs.
4. Test your 2FA by trying to connect to `ssh-forward.tuni.fi` from a non-university network (e.g. using your cell-phone internet and hotspot).
5. Try to connect to your machine, use `ssh -J your_tuni_username@ssh-forward.tuni.fi your_host_username@*********.pit.cs.tut.fi`
    - If you are on a University network (`roam.fi/eduroam/TUNI-STAFF` or VPN), it will only ask for your TUNI and host passwords;
    - If you are using a non-University network, it will first ask you for a `Verification code` which is a temporal code from `Google Authenticator` app or any other you installed on your smartphone.


!!! tip
    Config the `ssh` connection in `~/.ssh/config`:
    ``` python
    Host connection_name
      HostName ***********.pit.cs.tut.fi
      User your_username_at_the_host_machine
      ProxyCommand ssh your-tuni-username@ssh-forward.tuni.fi -W %h:%p
    ```
    After doing this, you will be able to do `ssh connection_name` to `ssh` directly to `*********.pit.cs.tut.fi`, forward ports, and transfer large files using `scp/rsync`. It is also useful if you are using `VSCode` or any other text editor which supports remote development. Additionally, it is handy if you would like to mount folders from the host to your client. You can use `sshfs connection_name:/path/to/remote_folder /path/to/local_folder`.


## How to Initialize the Two-step Verification Remotely

!!! warning "`linux-ssh.cc.tut.fi` and `ssh-forward.cc.tut.fi` will be deprecated by February 2021"
    *`linux-ssh.cc.tut.fi` and `ssh-forward.cc.tut.fi` will be decommissioned during the 2nd week of 2021 and `staff-linux.cc.tut.fi` in February 2021. Therefore, if neither of the three is working for you, try using a VPN or go to the university and initialize authentication from roam.fi/eduroam/otherWiFi. You may also ask a colleague-friend to do it for you if they have an access/vpn/at-uni.*

If you are setting up your connection remotely and you don't have an access to the university networks nor VPN at the moment, you still can do it. For this, you may use the old ssh-server if it still works for you `ssh-forward.cc.tut.fi`, otherwise you will need to apply for another TUT service. Proceed to [tut.fi/omatunnus](https://www.tut.fi/omatunnus) (yes, `tut` not `tuni`) -> `Services` -> `System Access` (wait 10 sec) -> search for `linux-ssh.cc.tut.fi` or `staff-linux.cc.tut.fi` depending on whether your are a student or a staff. When the access is granted (5 mins), `ssh` to one of them, and from there `ssh` to `ssh-forward.tuni.fi` -- no verification code will be asked as these servers are in the university network. Then, proceed with the initialization of the two-factor auth.
