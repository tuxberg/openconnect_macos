
## Configuring OpenConnect with SafeNet eToken 5110 on macOS Sonoma via PKCS#11

Making OpenConnect work with the SafeNet eToken 5110 on macOS Sonoma using the PKCS#11 vendor module can be challenging. As of January 7, 2025, there is no official support for the SafeNet eToken in OpenSC, so attempting to get it to work may not be worthwhile. 
While running `pkcs11-tool` with the `--module` parameter might give the impression of functionality, this is misleading: there is no OpenSC-compatible driver for the eToken card, and it will not function outside of the `pkcs11-tool` command line.

### Step-by-Step Guide

1. **Install p11-kit using Homebrew**:
   ```bash
   brew install p11-kit
   ```

2. **Install the SafeNet Authentication Client for macOS**:
   - Download from the official site.

3. **Load the vendor PKCS#11 library**:
   - Ensure that the library `libeTPkcs11.dylib` is loaded. You will need to obtain the PKCS#11 URL/URI for use with OpenConnect.
   - Confirm that `libcrypto.1.1.dylib` is accessible, as it is required by `libeTPkcs11.dylib`. Add the following line to your `.zshrc` file for both your local user and root:
     ```bash
     export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/opt/local/lib/openssl-1.1
     ```

4. **Create a configuration file**:
   - Create a file named `p11-kit-et.module` in `/usr/local/etc/pkcs11/modules/` with the following content:
     ```
     module: libeTPkcs11.dylib
     ```

5. **Copy the library**:
   - Copy `libeTPkcs11.dylib` from `/usr/local/lib/` to `/usr/local/Cellar/p11-kit/0.25.5/lib/pkcs11/`.

6. **Verify successful loading**:
   - Insert your smart card and check if `libeTPkcs11.dylib` loads correctly by running:
     ```bash
     p11-kit list-modules
     ```
   - The output should resemble:
     ```
     module: p11-kit-et
         path: /usr/local/Cellar/p11-kit/0.25.5/lib/pkcs11/libeTPkcs11.dylib
         uri: pkcs11:library-description=SafeNet%20eToken%20PKCS%2311;library-manufacturer=SafeNet%2C%20Inc.
         library-description: SafeNet eToken PKCS#11
         library-manufacturer: SafeNet, Inc.
         library-version: 10.8
         token: ******
             uri: pkcs11:model=eToken;manufacturer=SafeNet%2C%20Inc.;serial=*****;token=***
             manufacturer: SafeNet, Inc.
             model: eToken
             serial-number: ******
             hardware-version: 2.0
             flags:
                   rng
                   login-required
                   user-pin-initialized
                   dual-crypto-operations
                   token-initialized
     ```

7. **List objects on the token**:
   - Use `p11tool` to list objects stored on your token and identify your certificate's URL:
     ```bash
     p11tool --login --provider=/usr/local/lib/libeTPkcs11.dylib --list-all
     ```
   - Enter your token PIN when prompted and look for a URL formatted like this:
     ```
     URL: pkcs11:model=eToken;manufacturer=SafeNet%2C%20Inc.;serial=*******;token=*******;id=*********;object=***************;type=cert
     ```

8. **Install, configure, and run OpenConnect**:
   - Install OpenConnect with Homebrew:
     ```bash
     brew install openconnect
     ```
   - Edit `/usr/local/etc/vpnc/vpnc-script` to comment out the following line:
     ```bash
     ACTIVE_NETWORK_SERVICE=`networksetup -listnetworkserviceorder | grep -B 1 "$ACTIVE_INTERFACE" | head -n 1 | awk '/\([0-9]+\)/{ print }'|cut -d " " -f2-`
     ```
   - Add this line instead:
     ```bash
     ACTIVE_NETWORK_SERVICE="Wi-Fi"
     ```

9. **Run OpenConnect**:
   - Switch to superuser mode:
     ```bash
     sudo su -
     ```
   - Start a new Zsh session to ensure environment variables are correctly exported:
     ```bash
     zsh
     ```
   - Execute OpenConnect using the URL obtained from `p11tool`, along with your VPN server address and user agent (specified for Cisco VPN):
     ```bash
     openconnect -c 'pkcs11:model=eToken;manufacturer=SafeNet%2C%20Inc.;serial=****;token=*;id=*****;object=****' https://your.vpn.com --useragent=AnyConnect
     ```

10. **Disconnect safely**:
    - Remember to use Ctrl+C to disconnect (press only once) so that all changes to routes and nameservers can be reverted properly.

This updated guide should assist you in configuring OpenConnect with your SafeNet eToken on macOS Sonoma effectively!
