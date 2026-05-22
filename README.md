# terraform-az-fk-expressroute

This repository contains a reusable **Terraform/OpenTofu module** for deploying **Azure ExpressRoute connectivity primitives** such as **ExpressRoute circuits** and optional **Virtual Network Gateway connections**.

It is part of the **[FoggyKitchen.com training ecosystem](https://foggykitchen.com/courses-2/)** and serves as the Azure private interconnect edge building block for hybrid and multicloud connectivity patterns.

---

## 🎯 Purpose

The goal of this module is to provide a **clean, composable, and educational reference implementation** for Azure ExpressRoute edge connectivity:

- Focused on **ExpressRoute circuit and gateway connection resources**
- No hidden VNet, GatewaySubnet, or Virtual Network Gateway creation
- Designed to be composed with **terraform-az-fk-vng** and cloud-specific private connectivity modules

This is **not** a full landing zone replacement. It is a **connectivity edge module** intended for learning, reuse, and composition.

---

## ✨ What the module does

The module creates:

- Azure ExpressRoute circuit
- Optional Azure Virtual Network Gateway connection

The module intentionally does **not** create:

- Resource Groups
- Virtual Networks
- Gateway subnets
- Virtual Network Gateways
- Public IPs

Each of those concerns belongs in its own dedicated module or composition layer.

---

## 📂 Repository Structure

```bash
terraform-az-fk-expressroute/
├── main.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── LICENSE
└── README.md
```

---

## 🚀 Example Usage

```hcl
module "expressroute" {
  source = "git::https://github.com/mlinxfeld/terraform-az-fk-expressroute.git?ref=v0.1.1"

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

## ⚙️ Module Inputs

### Core inputs

| Variable | Type | Required | Description |
|--------|------|----------|-------------|
| `name` | `string` | ✅ | ExpressRoute circuit name |
| `resource_group_name` | `string` | ✅ | Resource group name |
| `location` | `string` | ✅ | Azure region |
| `service_provider_name` | `string` | ❌ | ExpressRoute service provider name |
| `peering_location` | `string` | ✅ | ExpressRoute peering location |
| `bandwidth_in_mbps` | `number` | ✅ | Provisioned circuit bandwidth in Mbps |
| `sku` | `object` | ❌ | ExpressRoute SKU definition |
| `allow_classic_operations` | `bool` | ❌ | Whether classic operations are allowed on the circuit |
| `tags` | `map(string)` | ❌ | Resource tags |

### Gateway connection objects

| Variable | Type | Required | Description |
|--------|------|----------|-------------|
| `gateway_connection` | `object` | ❌ | Optional Virtual Network Gateway connection object created only when enabled |

### Gateway connection object schema

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

## 📤 Outputs

| Output | Description |
|------|-------------|
| `circuit_id` | ExpressRoute circuit ID |
| `circuit_name` | ExpressRoute circuit name |
| `service_key` | ExpressRoute service key |
| `gateway_connection_id` | Gateway connection ID, if created |
| `gateway_connection_name` | Gateway connection name, if created |

---

## 🧠 Design Philosophy

- Explicit over implicit
- Small modules over monoliths
- Azure Virtual Network Gateway separated from ExpressRoute edge configuration
- Optimized for **learning, reuse, and composition**

This makes the module useful for:

- Azure-to-OCI private interconnect
- Azure partner connectivity foundations
- Multicloud private networking labs
- Progressive connectivity building blocks

---

## 📌 Notes

- This module focuses on Azure ExpressRoute edge primitives rather than full topologies
- Azure gateway creation should remain modeled in **terraform-az-fk-vng**
- The consuming stack should own staged orchestration when partner readiness matters

---

## 🌐 Learn More

Visit [FoggyKitchen.com](https://foggykitchen.com/) for Azure, multicloud, and Terraform/OpenTofu learning resources.

---

## 🪪 License

Licensed under the **Universal Permissive License (UPL), Version 1.0**.  
See [LICENSE](LICENSE) for details.
