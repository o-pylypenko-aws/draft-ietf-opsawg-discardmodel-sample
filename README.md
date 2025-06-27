# Juniper SLAX Script for draft-ietf-opsawg-discardmodel
# README.txt
## Overview
This SLAX script implements a subset of the IETF draft-ietf-opsawg-discardmodel for Juniper MX routers. It collects and categorizes packet discard statistics from the Packet Forwarding Engine (PFE) and maps them to the standardized categories defined in the IETF draft.  Interface-level statistics remain available through existing Juniper metrics.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
## IETF Draft Mapping
The script maps Juniper's PFE trap codes to these IETF categories:
### Top-Level Aggregates (IETF Model Compliance)
- policy/l3/packets             → Total policy-based discards
- errors/l3/rx/packets          → Total Layer 3 receive errors
- errors/internal/packets       → Total internal device errors
### Unintended Discards (Errors)
- ttl-expired       → errors/l3/ttl-expired       (TTL/Hop Limit expiry)
- no-route          → errors/l3/no-route          (No matching route)
- internal-error    → errors/internal/packets     (Generic hardware errors)
### Intended Discards (Policy)  
- policy-acl        → policy/l3/acl               (ACL policy drops)
- policy-nullroute  → policy/l3/null-route        (Null route policy)
### IETF Model Path Examples
- Policy totals: /device/discards/policy/l3/packets
- Specific errors: /device/discards/errors/l3/ttl-expired
- Internal errors: /device/discards/errors/internal/packets
## How It Works
1. Executes "show pfe statistics exceptions" command to access detailed hardware discard reasons
2. Uses lookup table to map 100+ internal trap codes to 6 internal categories (acl, errorlocal, errorrx, noroute, nullroute, ttl)
3. Calculates both specific error types AND required IETF aggregate counters
4. Generates IETF-compliant XML with proper hierarchy
## Installation & Usage
### Prerequisites
- Juniper MX router with SLAX support
- Administrative access to device
- PFE statistics collection enabled
### Installation
1. Copy script to router:
   ```
   scp draft-ietf-opsawg-discardmodel-sample.slax user@router:/var/db/scripts/op/
   ```
2. Configure script in Junos:
   ```
   set system scripts op file draft-ietf-opsawg-discardmodel-sample.slax
   commit
   ```
### Usage

#### Via NETCONF
Execute the script through NETCONF RPC calls:

**NETCONF RPC structure:**
```xml
<rpc>
  <op-script>
    <script>draft-ietf-opsawg-discardmodel-sample.slax</script>
  </op-script>
</rpc>
```

**Example NETCONF session:**
```bash
# Connect via NETCONF over SSH
ssh -s user@router netconf

# Send RPC request
<?xml version="1.0" encoding="UTF-8"?>
<rpc message-id="1">
  <op-script>
    <script>draft-ietf-opsawg-discardmodel-sample.slax</script>
  </op-script>
</rpc>
]]>]]>
```

**Using ncclient (Python):**
```python
from ncclient import manager

with manager.connect(host='router-ip', port=830, username='user', 
                    password='password', hostkey_verify=False) as m:
    rpc = '''
    <op-script>
      <script>draft-ietf-opsawg-discardmodel-sample.slax</script>
    </op-script>
    '''
    result = m.rpc(rpc)
    print(result)
```
### Sample Output
```xml
<device>
  <discards>
    <!-- Detailed error classification -->
    <errors>
      <l3>
        <rx>
          <packets>12453</packets>
        </rx>
        <ttl-expired>678</ttl-expired>
        <no-route>234</no-route>
      </l3>
      <internal>
        <packets>56</packets>
      </internal>
    </errors>
    
    <!-- Policy-based discards with aggregates -->
    <policy>
      <l3>
        <packets>1801</packets>
        <acl>1789</acl>
        <null-route>12</null-route>
      </l3>
    </policy>
  </discards>
</device>
```
### YANG Model Integration
The output structure aligns with the IETF YANG data model (device-level statistics):
```
module: ietf-packet-discard-reporting
  +--ro device
     +--ro discards
        +--ro errors
        |  +--ro l3
        |  |  +--ro rx
        |  |  |  +--ro packets             # Total receive errors
        |  |  +--ro ttl-expired            # TTL expiry discards
        |  |  +--ro no-route               # No route discards
        |  +--ro internal
        |     +--ro packets                # Total internal errors
        +--ro policy
           +--ro l3
              +--ro packets                # Total policy discards
              +--ro acl                    # ACL policy discards
              +--ro null-route             # Null route policy discards
```

## References
- IETF Draft: https://datatracker.ietf.org/doc/draft-ietf-opsawg-discardmodel/
- Juniper SLAX Guide: https://www.juniper.net/documentation/en_US/junos/topics/concept/junos-script-automation-slax-overview.html
- PFE Statistics: https://www.juniper.net/documentation/en_US/junos/topics/reference/command-summary/show-pfe-statistics.html
## License
 Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
 SPDX-License-Identifier: MIT-0
## Contact
For questions about the IETF draft refer to the OPSAWG working group mailing list: opsawg@ietf.org
