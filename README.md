# VMware PowerCLI Cheat Sheet

A comprehensive reference guide for VMware PowerCLI commands and operations.

## Table of Contents

- [Installation](#installation)
- [Connection Management](#connection-management)
- [Virtual Machine Operations](#virtual-machine-operations)
- [Host Management](#host-management)
- [Cluster Operations](#cluster-operations)
- [Storage Management](#storage-management)
- [Network Configuration](#network-configuration)
- [Snapshot Management](#snapshot-management)
- [Resource Management](#resource-management)
- [Reporting and Monitoring](#reporting-and-monitoring)

## Installation

```powershell
# Install PowerCLI from PowerShell Gallery
Install-Module -Name VMware.PowerCLI -Scope CurrentUser

# Update PowerCLI to the latest version
Update-Module -Name VMware.PowerCLI

# Check PowerCLI version
Get-Module -Name VMware.PowerCLI -ListAvailable

# Set invalid certificate action (if needed for lab environments)
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false
```

## Connection Management

```powershell
# Connect to vCenter Server
Connect-VIServer -Server vcenter.domain.com -User administrator@vsphere.local -Password 'password'

# Connect with credentials prompt
Connect-VIServer -Server vcenter.domain.com

# Connect to multiple vCenter servers
Connect-VIServer -Server vcenter1.domain.com,vcenter2.domain.com

# Disconnect from vCenter
Disconnect-VIServer -Server * -Confirm:$false

# Get current connections
$global:DefaultVIServers
```

## Virtual Machine Operations

### VM Information and Discovery

```powershell
# List all VMs
Get-VM

# Get specific VM
Get-VM -Name "VM-Name"

# Get VMs by pattern
Get-VM -Name "Web*"

# Get VM details
Get-VM -Name "VM-Name" | Select-Object Name, PowerState, NumCpu, MemoryGB, UsedSpaceGB

# Get VMs by folder
Get-Folder "Production" | Get-VM

# Get VMs by tag
Get-VM -Tag "Critical"
```

### Power Management

```powershell
# Power on VM
Start-VM -VM "VM-Name"

# Power on multiple VMs
Get-VM -Name "Web*" | Start-VM

# Power off VM (graceful shutdown)
Stop-VM -VM "VM-Name" -Confirm:$false

# Power off VM (hard stop)
Stop-VM -VM "VM-Name" -Kill -Confirm:$false

# Restart VM
Restart-VM -VM "VM-Name" -Confirm:$false

# Suspend VM
Suspend-VM -VM "VM-Name" -Confirm:$false
```

### VM Creation and Configuration

```powershell
# Create new VM
New-VM -Name "New-VM" -VMHost (Get-VMHost "esxi01.domain.com") -Datastore "Datastore1" -DiskGB 40 -MemoryGB 4 -NumCpu 2

# Create VM from template
New-VM -Name "New-VM" -Template "Windows2019-Template" -VMHost "esxi01.domain.com" -Datastore "Datastore1"

# Clone VM
New-VM -Name "VM-Clone" -VM "Source-VM" -VMHost "esxi01.domain.com" -Datastore "Datastore1"

# Modify VM CPU
Set-VM -VM "VM-Name" -NumCpu 4 -Confirm:$false

# Modify VM memory
Set-VM -VM "VM-Name" -MemoryGB 8 -Confirm:$false

# Add hard disk
New-HardDisk -VM "VM-Name" -CapacityGB 100 -StorageFormat Thin

# Remove VM
Remove-VM -VM "VM-Name" -DeletePermanently -Confirm:$false
```

### Guest OS Operations

```powershell
# Get VMware Tools status
Get-VM | Select-Object Name, @{N="Tools Status";E={$_.ExtensionData.Guest.ToolsStatus}}

# Update VMware Tools
Update-Tools -VM "VM-Name"

# Get guest information
Get-VMGuest -VM "VM-Name"

# Execute script in guest OS (requires guest credentials)
Invoke-VMScript -VM "VM-Name" -ScriptText "ipconfig" -GuestUser "Administrator" -GuestPassword "password"
```

## Host Management

```powershell
# List all ESXi hosts
Get-VMHost

# Get host details
Get-VMHost -Name "esxi01.domain.com" | Select-Object Name, ConnectionState, PowerState, Version

# Get host hardware information
Get-VMHost -Name "esxi01.domain.com" | Get-VMHostHardware

# Enter maintenance mode
Set-VMHost -VMHost "esxi01.domain.com" -State Maintenance -Confirm:$false

# Exit maintenance mode
Set-VMHost -VMHost "esxi01.domain.com" -State Connected -Confirm:$false

# Restart ESXi host
Restart-VMHost -VMHost "esxi01.domain.com" -Confirm:$false

# Get host NTP configuration
Get-VMHost | Get-VMHostNtpServer

# Set NTP server
Add-VmHostNtpServer -VMHost "esxi01.domain.com" -NtpServer "pool.ntp.org"

# Get host services
Get-VMHost "esxi01.domain.com" | Get-VMHostService
```

## Cluster Operations

```powershell
# List all clusters
Get-Cluster

# Get cluster details
Get-Cluster -Name "Production-Cluster" | Select-Object Name, HAEnabled, DrsEnabled, DrsAutomationLevel

# Enable HA on cluster
Set-Cluster -Cluster "Production-Cluster" -HAEnabled:$true -Confirm:$false

# Enable DRS on cluster
Set-Cluster -Cluster "Production-Cluster" -DrsEnabled:$true -DrsAutomationLevel FullyAutomated -Confirm:$false

# Add host to cluster
Add-VMHost -Name "esxi03.domain.com" -Location "Production-Cluster" -User root -Password "password"

# Get cluster resource information
Get-Cluster "Production-Cluster" | Select-Object Name, @{N="Total CPU (GHz)";E={[math]::Round($_.ExtensionData.Summary.TotalCpu/1000,2)}}, @{N="Total Memory (GB)";E={[math]::Round($_.ExtensionData.Summary.TotalMemory/1GB,2)}}
```

## Storage Management

### Datastores

```powershell
# List all datastores
Get-Datastore

# Get datastore details
Get-Datastore | Select-Object Name, Type, CapacityGB, FreeSpaceGB, @{N="Used%";E={[math]::Round(($_.CapacityGB - $_.FreeSpaceGB)/$_.CapacityGB*100,2)}}

# Get VMs on specific datastore
Get-Datastore "Datastore1" | Get-VM

# Increase datastore capacity
Set-Datastore -Datastore "Datastore1" -CapacityGB 2000

# Create new NFS datastore
New-Datastore -Nfs -VMHost "esxi01.domain.com" -Name "NFS-Datastore" -Path "/export/nfs" -NfsHost "nfs-server.domain.com"
```

### Storage vMotion

```powershell
# Migrate VM to different datastore
Move-VM -VM "VM-Name" -Datastore "Datastore2"

# Migrate VM to different datastore and host
Move-VM -VM "VM-Name" -Destination (Get-VMHost "esxi02.domain.com") -Datastore "Datastore2"
```

## Network Configuration

### Virtual Switches

```powershell
# Get virtual switches
Get-VirtualSwitch

# Get standard switch details
Get-VirtualSwitch -Standard

# Get distributed switch details
Get-VDSwitch

# Create standard virtual switch
New-VirtualSwitch -VMHost "esxi01.domain.com" -Name "vSwitch2"

# Add port group to standard switch
New-VirtualPortGroup -VirtualSwitch "vSwitch2" -Name "Production-VLAN100" -VLanId 100
```

### Network Adapters

```powershell
# Get VM network adapters
Get-VM "VM-Name" | Get-NetworkAdapter

# Add network adapter to VM
New-NetworkAdapter -VM "VM-Name" -NetworkName "Production-VLAN100" -StartConnected -Type Vmxnet3

# Modify network adapter
Set-NetworkAdapter -NetworkAdapter (Get-VM "VM-Name" | Get-NetworkAdapter) -NetworkName "New-PortGroup" -Confirm:$false

# Remove network adapter
Remove-NetworkAdapter -NetworkAdapter (Get-VM "VM-Name" | Get-NetworkAdapter -Name "Network adapter 2") -Confirm:$false
```

## Snapshot Management

```powershell
# Create snapshot
New-Snapshot -VM "VM-Name" -Name "Before-Update" -Description "Snapshot before patching" -Memory:$false -Quiesce:$true

# List all snapshots
Get-VM | Get-Snapshot

# Get snapshots for specific VM
Get-VM "VM-Name" | Get-Snapshot

# Revert to snapshot
Set-VM -VM "VM-Name" -Snapshot (Get-Snapshot -VM "VM-Name" -Name "Before-Update") -Confirm:$false

# Remove snapshot
Remove-Snapshot -Snapshot (Get-Snapshot -VM "VM-Name" -Name "Before-Update") -Confirm:$false

# Remove all snapshots from VM
Get-Snapshot -VM "VM-Name" | Remove-Snapshot -Confirm:$false

# Find VMs with snapshots older than 7 days
Get-VM | Get-Snapshot | Where-Object {$_.Created -lt (Get-Date).AddDays(-7)} | Select-Object VM, Name, Created, SizeGB
```

## Resource Management

### Resource Pools

```powershell
# Get resource pools
Get-ResourcePool

# Create resource pool
New-ResourcePool -Location "Production-Cluster" -Name "WebServers" -CpuSharesLevel High -MemSharesLevel High

# Move VM to resource pool
Move-VM -VM "VM-Name" -Destination (Get-ResourcePool "WebServers")

# Set resource pool limits
Set-ResourcePool -ResourcePool "WebServers" -CpuLimitMhz 10000 -MemLimitGB 64
```

### vMotion

```powershell
# Migrate VM to another host (vMotion)
Move-VM -VM "VM-Name" -Destination (Get-VMHost "esxi02.domain.com")

# Migrate multiple VMs
Get-VM -Location "esxi01.domain.com" | Move-VM -Destination (Get-VMHost "esxi02.domain.com")
```

## Reporting and Monitoring

### Performance Monitoring

```powershell
# Get VM performance statistics
Get-Stat -Entity (Get-VM "VM-Name") -Stat cpu.usage.average,mem.usage.average -Start (Get-Date).AddDays(-1)

# Get host performance statistics
Get-Stat -Entity (Get-VMHost "esxi01.domain.com") -Stat cpu.usage.average,mem.usage.average -IntervalMins 5 -MaxSamples 12

# Get real-time CPU usage
Get-VM | Select-Object Name, @{N="CPU Usage (%)";E={[math]::Round((Get-Stat -Entity $_ -Stat cpu.usage.average -Realtime -MaxSamples 1).Value,2)}}
```

### Inventory Reports

```powershell
# Export VM inventory to CSV
Get-VM | Select-Object Name, PowerState, NumCpu, MemoryGB, @{N="UsedSpaceGB";E={[math]::Round($_.UsedSpaceGB,2)}}, @{N="ProvisionedSpaceGB";E={[math]::Round($_.ProvisionedSpaceGB,2)}} | Export-Csv -Path "VM-Inventory.csv" -NoTypeInformation

# Get host inventory
Get-VMHost | Select-Object Name, ConnectionState, PowerState, Version, Build, Manufacturer, Model | Export-Csv -Path "Host-Inventory.csv" -NoTypeInformation

# Get datastore usage report
Get-Datastore | Select-Object Name, Type, @{N="CapacityGB";E={[math]::Round($_.CapacityGB,2)}}, @{N="FreeSpaceGB";E={[math]::Round($_.FreeSpaceGB,2)}}, @{N="UsedSpaceGB";E={[math]::Round($_.CapacityGB - $_.FreeSpaceGB,2)}} | Export-Csv -Path "Datastore-Report.csv" -NoTypeInformation
```

### Alerts and Events

```powershell
# Get recent events
Get-VIEvent -MaxSamples 100

# Get events for specific VM
Get-VIEvent -Entity (Get-VM "VM-Name") -MaxSamples 50

# Get error events
Get-VIEvent -Types Error -Start (Get-Date).AddDays(-1)

# Get VM creation events
Get-VIEvent -Types Info | Where-Object {$_.FullFormattedMessage -like "*created*"}
```

---

## Additional Resources

- [VMware PowerCLI Documentation](https://developer.vmware.com/powercli)
- [PowerCLI Community](https://communities.vmware.com/t5/VMware-PowerCLI/ct-p/2006)
- [PowerCLI Blog](https://blogs.vmware.com/PowerCLI/)

## License

This cheat sheet is provided as-is for educational and reference purposes.
