Title: Why Your Azure Spot VM Keeps Going Down
Date: 2026-09-10
Category: Infrastructure
Tags: azure, spot-vms, cloud, devops
Slug: why-your-azure-spot-vm-keeps-going-down
Cover: images/why-your-azure-spot-vm-keeps-going-down.png

If your Azure Spot VM keeps dropping, it almost always comes down to how you configured its eviction type and policy when you created it. Spot VMs give you up to 90% off pay-as-you-go pricing by using Azure's unused capacity, but that discount comes with a real chance of interruption. Here's what's actually causing it, and how to control it.

## Eviction Type: Capacity-Only vs. Price or Capacity
This is the setting you choose at VM creation time, and it decides *why* Azure can take your VM away.
With **capacity-only**, your VM is only evicted for capacity reasons, not price reasons, which maximizes the time your VM stays running. With **price or capacity**, Azure adds a second trigger: an eviction happens when excess compute capacity is no longer available or when the cost of the VM exceeds the maximum price you set. If you didn't deliberately pick capacity-only, you may be getting evicted over price swings without realizing it.

## Capacity Reclaim Can't Be Avoided by Price Settings
This is the one people miss most often. When Azure detects insufficient capacity in a region for standard VM demand, it reclaims Spot VM capacity — and this happens regardless of your max price setting. Even a VM configured to never evict on price will still go down if Azure needs the hardware back for higher-priority pay-as-you-go workloads. Capacity-based evictions are the most common cause of Spot VM loss in heavily used regions, and the only real mitigation is architectural.

## Eviction Policy: Deallocate vs. Delete
Separate from eviction *type*, this setting decides what happens to the VM once it's evicted.
The **Deallocate** policy (the default) moves your VM to the stopped-deallocated state so you can redeploy it later — though there's no guarantee reallocation will succeed, and you'll still be charged storage costs for the underlying disks. The **Delete** policy is more aggressive: the VM and all associated resources, including disks, NICs, and public IPs, are deleted outright. If you're seeing your VM and its resources vanish entirely rather than just stopping, check this setting first.

## The 30-Second Warning
Azure doesn't pull the plug with zero notice. A Spot VM can be evicted with thirty seconds' notice, delivered through the Scheduled Events API. That's little time, but it's enough to checkpoint in-progress work, drain connections, or push a final log write — if your app is actually listening for it.

## Getting Reallocated After Eviction
What happens next also depends on your setup. If your Spot VM is evicted because of capacity and is part of a scale set, Azure will try to restore it to maintain your target instance count. A single standalone Spot VM, however, is always deallocated by Azure and won't come back on its own — you have to manually restart it, and reallocation still depends on capacity being available again.

**How to actually reduce the churn:** run Spot VMs in a scale set instead of standalone where possible, listen for the Scheduled Events warning and checkpoint accordingly, and pick capacity-only eviction if unpredictable pricing evictions are the thing hurting you most.