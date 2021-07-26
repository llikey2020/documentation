# AWS EKS NVMe research

In July 2021, it is not possible to run AWS EKS using instances with NVMe devices and make use of those devices.
The problem is that k8s emptyDir does not allow the physical path to be specified. Therefore there is no way to get k8s to use the NVMe drives.
