﻿---
title: 'Connect VPN gateways to multiple on-premises policy-based VPN devices'
titleSuffix: Azure VPN Gateway
description: Learn how to configure an Azure route-based VPN gateway to multiple policy-based VPN devices using PowerShell.
services: vpn-gateway
author: yushwang

ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang

---
# Connect Azure VPN gateways to multiple on-premises policy-based VPN devices using PowerShell

This article helps you configure an Azure route-based VPN gateway to connect to multiple on-premises policy-based VPN devices leveraging custom IPsec/IKE policies on S2S VPN connections.

## <a name="about"></a>About policy-based and route-based VPN gateways

Policy-based *vs.* route-based VPN devices differ in how the IPsec traffic selectors are set on a connection:

* **Policy-based** VPN devices use the combinations of prefixes from both networks to define how traffic is encrypted/decrypted through IPsec tunnels. It is typically built on firewall devices that perform packet filtering. IPsec tunnel encryption and decryption are added to the packet filtering and processing engine.
* **Route-based** VPN devices use any-to-any (wildcard) traffic selectors, and let routing/forwarding tables direct traffic to different IPsec tunnels. It is typically built on router platforms where each IPsec tunnel is modeled as a network interface or VTI (virtual tunnel interface).

The following diagrams highlight the two models:

### Policy-based VPN example
![policy-based](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedmultisite.png)

### Route-based VPN example
![route-based](./media/vpn-gateway-connect-multiple-policybased-rm-ps/routebasedmultisite.png)

### Azure support for policy-based VPN
Currently, Azure supports both modes of VPN gateways: route-based VPN gateways and policy-based VPN gateways. They are built on different internal platforms, which result in different specifications:

| Category | PolicyBased VPN Gateway | RouteBased VPN Gateway | RouteBased VPN Gateway |  RouteBased VPN Gateway
| -------- | ----------------------- | ---------------------- | ---------------------- | ----------------------- |
| **Azure Gateway SKU**    | Basic                       | Basic                            | VpnGw1, VpnGw2, VpnGw3  | VpnGw4 and VpnGw5 |
| **IKE version**          | IKEv1                       | IKEv2                            | IKEv1 and IKEv2         | IKEv1 and IKEv2   |
| **Max. S2S connections** | **1**                       | 10                               | 30                      | 100               |
|                          |                             |                                  |                         |                   |

With the custom IPsec/IKE policy, you can now configure Azure route-based VPN gateways to use prefix-based traffic selectors with option "**PolicyBasedTrafficSelectors**", to connect to on-premises policy-based VPN devices. This capability allows you to connect from an Azure virtual network and VPN gateway to multiple on-premises policy-based VPN/firewall devices, removing the single connection limit from the current Azure policy-based VPN gateways.

> [!IMPORTANT]
> 1. To enable this connectivity, your on-premises policy-based VPN devices must support **IKEv2** to connect to the Azure route-based VPN gateways. Check your VPN device specifications.
> 2. The on-premises networks connecting through policy-based VPN devices with this mechanism can only connect to the Azure virtual network; **they cannot transit to other on-premises networks or virtual networks via the same Azure VPN gateway**.
> 3. The configuration option is part of the custom IPsec/IKE connection policy. If you enable the policy-based traffic selector option, you must specify the complete policy (IPsec/IKE encryption and integrity algorithms, key strengths, and SA lifetimes).

The following diagram shows why transit routing via Azure VPN gateway doesn't work with the policy-based option:

![policy-based transit](./media/vpn-gateway-connect-multiple-policybased-rm-ps/policybasedtransit.png)

As shown in the diagram, the Azure VPN gateway has traffic selectors from the virtual network to each of the on-premises network prefixes, but not the cross-connection prefixes. For example, on-premises site 2, site 3, and site 4 can each communicate to VNet1 respectively, but cannot connect via the Azure VPN gateway to each other. The diagram shows the cross-connect traffic selectors that are not available in the Azure VPN gateway under this configuration.

## <a name="workflow"></a>Workflow

The instructions in this article follow the same example as described in [Configure IPsec/IKE policy for S2S or VNet-to-VNet connections](vpn-gateway-ipsecikepolicy-rm-powershell.md) to establish a S2S VPN connection. This is shown in the following diagram:

![s2s-policy](./media/vpn-gateway-connect-multiple-policybased-rm-ps/s2spolicypb.png)

The workflow to enable this connectivity:
1. Create the virtual network, VPN gateway, and local network gateway for your cross-premises connection.
2. Create an IPsec/IKE policy.
3. Apply the policy when you create a S2S or VNet-to-VNet connection, and **enable the policy-based traffic selectors** on the connection.
4. If the connection is already created, you can apply or update the policy to an existing connection.

## Before you begin

* Verify that you have an Azure subscription. If you don't already have an Azure subscription, you can activate your [MSDN subscriber benefits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) or sign up for a [free account](https://azure.microsoft.com/pricing/free-trial).

* [!INCLUDE [powershell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="enablepolicybased"></a>Enable policy-based traffic selectors

This section shows you how to enable policy-based traffic selectors on a connection. Make sure you have completed [Part 3 of the Configure IPsec/IKE policy article](vpn-gateway-ipsecikepolicy-rm-powershell.md). The steps in this article use the same parameters.

### Step 1 - Create the virtual network, VPN gateway, and local network gateway

#### Connect to your subscription and declare your variables

1. If you are running PowerShell locally on your computer, sign in using the *Connect-AzAccount* cmdlet. Or, instead, use Azure Cloud Shell in your browser.

2. Declare your variables. For this exercise, we use the following variables:

   ```azurepowershell-interactive
   $Sub1          = "<YourSubscriptionName>"
   $RG1           = "TestPolicyRG1"
   $Location1     = "East US 2"
   $VNetName1     = "TestVNet1"
   $FESubName1    = "FrontEnd"
   $BESubName1    = "Backend"
   $GWSubName1    = "GatewaySubnet"
   $VNetPrefix11  = "10.11.0.0/16"
   $VNetPrefix12  = "10.12.0.0/16"
   $FESubPrefix1  = "10.11.0.0/24"
   $BESubPrefix1  = "10.12.0.0/24"
   $GWSubPrefix1  = "10.12.255.0/27"
   $DNS1          = "8.8.8.8"
   $GWName1       = "VNet1GW"
   $GW1IPName1    = "VNet1GWIP1"
   $GW1IPconf1    = "gw1ipconf1"
   $Connection16  = "VNet1toSite6"
   $LNGName6      = "Site6"
   $LNGPrefix61   = "10.61.0.0/16"
   $LNGPrefix62   = "10.62.0.0/16"
   $LNGIP6        = "131.107.72.22"
   ```

#### Create the virtual network, VPN gateway, and local network gateway

1. Create a resource group.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG1 -Location $Location1
   ```
2. Use the following example to create the virtual network TestVNet1 with three subnets, and the VPN gateway. If you want to substitute values, it's important that you always name your gateway subnet specifically 'GatewaySubnet'. If you name it something else, your gateway creation fails.

    ```azurepowershell-interactive
    $fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
    $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
    $gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1
    
    New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
    
    $gw1pip1    = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
    $vnet1      = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
    $subnet1    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
    $gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1
    
    New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance
    
    New-AzLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
    ```

### Step 2 - Create an S2S VPN connection with an IPsec/IKE policy

1. Create an IPsec/IKE policy.

   > [!IMPORTANT]
   > You need to create an IPsec/IKE policy in order to enable "UsePolicyBasedTrafficSelectors" option
   > on the connection.

   The following example creates an IPsec/IKE policy with these algorithms and parameters:
    * IKEv2: AES256, SHA384, DHGroup24
    * IPsec: AES256, SHA256, PFS None, SA Lifetime 14400 seconds & 102400000KB

   ```azurepowershell-interactive
   $ipsecpolicy6 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
   ```
1. Create the S2S VPN connection with policy-based traffic selectors and IPsec/IKE policy and apply the IPsec/IKE policy created in the previous step. Be aware of the additional parameter "-UsePolicyBasedTrafficSelectors $True", which enables policy-based traffic selectors on the connection.

   ```azurepowershell-interactive
   $vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
   $lng6 = Get-AzLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

   New-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -UsePolicyBasedTrafficSelectors $True -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
   ```
1. After completing the steps, the S2S VPN connection will use the IPsec/IKE policy defined, and enable policy-based traffic selectors on the connection. You can repeat the same steps to add more connections to additional on-premises policy-based VPN devices from the same Azure VPN gateway.

## <a name="update"></a>To update policy-based traffic selectors
This section shows you how to update the policy-based traffic selectors option for an existing S2S VPN connection.

1. Get the connection resource.

   ```azurepowershell-interactive
   $RG1          = "TestPolicyRG1"
   $Connection16 = "VNet1toSite6"
   $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
   ```
1. View the policy-based traffic selectors option.
The following line shows whether the policy-based traffic selectors are used for the connection:

   ```azurepowershell-interactive
   $connection6.UsePolicyBasedTrafficSelectors
   ```

   If the line returns "**True**", then policy-based traffic selectors are configured on the connection; otherwise it returns "**False**."
1. Once you obtain the connection resource, you can enable or disable the policy-based traffic selectors on a connection.

   - To Enable

      The following example enables the policy-based traffic selectors option, but leaves the IPsec/IKE policy unchanged:

      ```azurepowershell-interactive
      $RG1          = "TestPolicyRG1"
      $Connection16 = "VNet1toSite6"
      $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

      Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $True
      ```

   - To Disable

      The following example disables the policy-based traffic selectors option, but leaves the IPsec/IKE policy unchanged:

      ```azurepowershell-interactive
      $RG1          = "TestPolicyRG1"
      $Connection16 = "VNet1toSite6"
      $connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

      Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -UsePolicyBasedTrafficSelectors $False
      ```

## Next steps
Once your connection is complete, you can add virtual machines to your virtual networks. See [Create a Virtual Machine](../virtual-machines/windows/quick-create-portal.md) for steps.

Also review [Configure IPsec/IKE policy for S2S VPN or VNet-to-VNet connections](vpn-gateway-ipsecikepolicy-rm-powershell.md) for more details on custom IPsec/IKE policies.
