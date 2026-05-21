# terraform-az-fk-expressroute

This repository contains a reusable **Terraform / OpenTofu module** for deploying an **Azure ExpressRoute circuit** and, when explicitly enabled, the **Virtual Network Gateway connection** that binds the circuit to an Azure VNet gateway.

---

## Purpose

The module is designed for composable Azure connectivity patterns where:

- the ExpressRoute circuit lifecycle is managed separately from the VNet gateway
- the final gateway connection may need to be delayed until the remote side is operational
- the consuming stack owns the surrounding VNet, GatewaySubnet, and VNG resources

This makes it a natural fit for staged OCI FastConnect interconnect deployments.

---

## What the module does

The module creates:

- one `azurerm_express_route_circuit`
- optional `azurerm_virtual_network_gateway_connection`

The module intentionally does **not** create:

- Resource Groups
- Virtual Networks
- Gateway subnets
- Virtual Network Gateways

Those resources should be composed from separate reusable modules.

---

## Example Usage

```hcl
module "expressroute" {
  source = "git::https://github.com/mlinxfeld/terraform-az-fk-expressroute.git?ref=v0.1.0"

  name                = "er-fk-demo"
  location            = "westeurope"
  resource_group_name = "rg-fk-demo"
  peering_location    = "Amsterdam"
  bandwidth_in_mbps   = 1000

  gateway_connection = {
    enabled                    = false
    name                       = "conn-fk-demo"
    virtual_network_gateway_id = module.vng.gateway_id
  }
}
```

Enable the connection only after the remote provider side is ready, then apply again.

---

## Inputs

| Variable | Required | Description |
|------|------|-------------|
| `name` | ✅ | ExpressRoute circuit name |
| `resource_group_name` | ✅ | Resource group name |
| `location` | ✅ | Azure region |
| `service_provider_name` | ❌ | Service provider name |
| `peering_location` | ✅ | ExpressRoute peering location |
| `bandwidth_in_mbps` | ✅ | Circuit bandwidth |
| `sku` | ❌ | SKU object with `tier` and `family` |
| `allow_classic_operations` | ❌ | Allow classic operations |
| `gateway_connection` | ❌ | Optional staged VNG connection object |
| `tags` | ❌ | Resource tags |

### Gateway connection schema

```hcl
gateway_connection = object({
  enabled                      = optional(bool, false)
  name                         = string
  virtual_network_gateway_id   = string
  authorization_key            = optional(string)
  routing_weight               = optional(number)
  express_route_gateway_bypass = optional(bool, false)
})
```

---

## Outputs

| Output | Description |
|------|-------------|
| `circuit_id` | ExpressRoute circuit ID |
| `circuit_name` | ExpressRoute circuit name |
| `service_key` | ExpressRoute service key |
| `gateway_connection_id` | Gateway connection ID, if created |
| `gateway_connection_name` | Gateway connection name, if created |

---

## License

Licensed under the **Universal Permissive License (UPL), Version 1.0**.
