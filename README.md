Kerberos-Attestor
==

Overview
--

The Kerberos-Attestor is a plugin for the [SPIRE][spire] server and agent that allows SPIRE to automatically attest nodes that are joined to a domain backed by the [Kerberos authentication protocol][kerberos].  SPIRE is an open-source implementation of the [SPIFFE][spiffe], which is a set of standards to provide authentication and trust to disparate micro-services operating in heterogeneous cloud-native environments.  The predominant on-premise authentication protocol is Kerberos through Active Directory, and with the Kerberos-Attestor, environments backed by SPIRE can provide trust leveraging existing enterprise identity stacks.

Base SVID SPIFFE ID Format
--

An agent attested by the Kerberos-Attestor plugin will have a base SVID SPIFFE ID in this format:

  `spiffe://<trust_domain>/spire/agent/kerberos/<realm>/<principal_name>`

Compilation
--

There are two ways to get the plugin--using `go install` to build and install it or alternatively, build it from source.

**Go Install**

Running the following commands will download, build, and install the Kerberos-Attestor server and agent in your `${GOPATH}/bin` directory by default, or in the path set by the `${GOBIN}` environment variable.

* Server:
  * `go install github.com/spiffe/kerberos-attestor/server`
* Agent:
  * `go install github.com/spiffe/kerberos-attestor/agent`

**Build from Source**

1. Clone this repo:

  ```bash
  git clone https://github.com/spiffe/kerberos-attestor ${GOPATH}/src/github.com/spiffe/kerberos-attestor
  cd ${GOPATH}/src/github.com/spiffe/kerberos-attestor
  ```

2. Build the Kerberos-Attestor:

  ```bash
  make build
  ```

3. Binaries for the server and agent should be in the `bin/` directory

Installation and Configuration
--

**Kerberos-Attestor Server Plugin**

1. Edit the SPIRE Server config file to add the Kerberos-Attestor server plugin config:

  ```bash
  vim <SPIRE Installation Directory>/conf/server/server.conf
  ```

2. Add the following [HCL][hcl] blob to the "plugins" section of the config file:

  ```bash
  NodeAttestor "kerberos_attestor" {
      plugin_cmd = "${GOPATH}/src/github.com/spiffe/kerberos-attestor/bin/server"
      enabled = true
      plugin_data {
          krb_realm = "<realm>"
          krb_conf_path = "/etc/krb5.conf"
          krb_keytab_path = "/etc/krb5.keytab"
      }
  }
  ```
  * Replace `plugin_cmd` with the path to the Kerberos-Attestor server binary compiled earlier
  * Replace `krb_realm` with the real Kerberos realm
  * `krb_conf_path` and `krb_keytab_path` point to the default paths to the Kerberos config file and Keytab file

**Kerberos-Attestor Agent Plugin**

1. Edit the SPIRE Agent config file to add the Kerberos-Attestor agent plugin config:

  ```bash
  vim <SPIRE Installation Directory>/conf/agent/agent.conf
  ```

2. Add the following [HCL][hcl] blob to the "plugins" section of the config file:

  ```bash
  NodeAttestor "kerberos_attestor" {
      plugin_cmd = "${GOPATH}/src/github.com/spiffe/kerberos-attestor/bin/agent"
      enabled = true
      plugin_data {
          krb_realm = "<realm>"
          krb_conf_path = "/etc/krb5.conf"
          krb_keytab_path = "/etc/krb5.keytab"
          spn = "<service principal name>"
      }
  }
  ```
  * Replace `plugin_cmd` with the path to the Kerberos-Attestor agent binary compiled earlier
  * Replace `krb_realm` with the real Kerberos realm
  * `krb_conf_path` and `krb_keytab_path` point to the default paths to the Kerberos config file and Keytab file
  * Replace `spn` with the real Service Principal Name of the keytab file of the Kerberos attestor server

3. Remove Join-Token NodeAttestor plugin config from this file as a SPIRE Agent can only use one NodeAttestor plugin at-a-time

Start SPIRE with Kerberos-Attestor Plugins
--

**SPIRE Server**

```bash
cd <SPIRE Installation Directory>
./spire-server run
```

**SPIRE Agent**

```bash
cd <SPIRE Installation Directory>
./spire-agent run
```

Selectors Generated by Kerberos-Attestor Server Plugin
--

| Selector   | Example             | Description             |
| ---------- | ------------------- | ----------------------- |
| pn         | `pn:host/216.3.128.12` | Principal name of the agent |


[spire]: https://github.com/spiffe/spire
[spiffe]: https://github.com/spiffe/spiffe
[kerberos]: https://en.wikipedia.org/wiki/Kerberos_(protocol)
[hcl]: https://github.com/hashicorp/hcl
