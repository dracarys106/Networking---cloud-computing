# Networking---cloud-computing

Navigation Menu
Networks-Cloud-Computing

Networking Fundamentals (14/10/25)
TUN (TAP Unicast Network)
Definition: Virtual network interface that operates at Layer 3 (Network layer).

Key Points:

Packets are read/written as complete IP packets in user space
Operates on IP datagrams (not raw frames)
Single End Point mode (unicast only)
Used for VPN applications, tunneling, and routing software
Lower overhead than TAP for IP-only applications
No Ethernet headers present
Common in applications like OpenVPN, WireGuard
Use Case: Ideal when you need to process IP packets without dealing with Ethernet frame details.

Tap() - TAP (Terminal Access Point)
Definition: Virtual network interface operating at Layer 2 (Data Link layer).

Key Points:

Packets are read/written as complete Ethernet frames
Full frame including Ethernet header, payload, and trailer
Supports ARP, DHCP, and other Layer 2 protocols
Can simulate a physical network interface card
Bridge between user space and kernel network stack
Supports broadcast and multicast traffic
Higher flexibility but slightly higher overhead than TUN
Use Case: Virtual machine hypervisors (KVM, QEMU), network simulation, packet manipulation tools.

OVS (Open vSwitch)
Definition: Multilayer virtual switch enabling network automation through programmatic extension.

Key Points:

Open-source software implementation of a virtual switch
Implements OpenFlow standard for network control
Operates at Layer 2-3, supporting both bridging and routing
Kernel module (openvswitch.ko) handles data plane
User space daemon (ovsdb-server, ovs-vswitchd) handles control plane
Supports VLANs, spanning tree, network QoS
Performance optimized with flow caching and kernel offloading
Works with Linux bridges and provides more advanced features
Commonly used in cloud platforms (OpenStack) and container orchestration
Architecture: User space control plane manages policies; kernel module fast-paths actual packet forwarding.

VXLAN (Virtual Extensible LAN)
Definition: Layer 2 over Layer 3 encapsulation protocol enabling large-scale network virtualization.

Key Points:

Encapsulates Ethernet frames in UDP packets (MAC-in-UDP)
Header: Outer IP + UDP + VXLAN header + Original Ethernet frame
24-bit VXLAN Network Identifier (VNI) supports 16 million isolated networks
UDP port 4789 (IANA assigned)
Enables multi-tenant environments with 4094+ VLANs
Network underlays can be any IP infrastructure (physical, cloud, hybrid)
Addresses VLAN limitations (12-bit tag = 4094 VLANs only)
Requires VXLAN tunnel endpoints (VTEPs) to encapsulate/decapsulate traffic
Reduces dependency on physical network configuration
Learning via source MAC address or control plane (multicast or unicast)
Advantages: Overcomes VLAN limitations, enables network mobility, supports overlay networks.

Flannel (CNI)
Definition: Simple, lightweight Container Network Interface plugin by CoreOS for Kubernetes networking.

Key Points:

Provides Layer 3 overlay network for Kubernetes clusters
Each node gets a subnet (e.g., 10.1.x.0/24) to assign to pods
Multiple backend options: VXLAN, UDP, host-gw, AWS VPC
VXLAN backend: Encapsulates pod traffic across nodes
host-gw backend: Direct routes without encapsulation (requires Layer 2 connectivity)
UDP backend: Simple but slower, used primarily for development
Subnet allocation managed by etcd (distributed consensus)
Pod-to-pod communication via overlay network spanning cluster
Simple deployment and minimal resource footprint
Good for homogeneous clusters with standard networking needs
Cannot enforce network policies (no built-in security policies)
Architecture: Flannel daemon runs on each node, manages routes and tunnel endpoints.

Cilium
Definition: eBPF-based networking and security solution providing network policies, load balancing, and observability for Kubernetes.

Key Points:

Uses eBPF (extended Berkeley Packet Filter) for high-performance packet processing
Operates at kernel level, bypassing traditional iptables
Provides L3/L4/L7 network policies with fine-grained control
Advanced load balancing with session affinity and direct server return
Multi-cluster networking and mesh capabilities
Integrates with service mesh (Istio, Linkerd)
API-aware policies at application layer (HTTP, gRPC, Kafka)
Flow visibility and observability at network packet level
Hubble component provides network flow visualization
XDP (Express Data Path) support for early packet processing
Can replace kube-proxy with more efficient implementation
Zero-trust security model with default-deny policies
Advantage: Superior performance and advanced security compared to iptables-based solutions.

Packet Copy
Definition: Data transfer mechanism where packet data is duplicated from one location to another.

Key Points:

Source buffer copied to destination buffer entirely
Traditional approach in network stack (read → copy → write)
Multiple copies occur: NIC → kernel → user space → kernel → NIC
Each copy consumes CPU cycles and memory bandwidth
Latency increases proportionally with packet size
Simple and reliable but resource-intensive
Default behavior in most operating systems
Suitable for low-throughput applications where latency acceptable
Creates memory overhead and cache pressure
Context: Occurs when kernel cannot directly share packet memory with application.

Zero Copy
Definition: Data transfer technique avoiding redundant copying, moving pointers instead of data.

Key Points:

Data remains in single memory location; pointers/references passed instead
Kernel and application share virtual memory mappings
DMA (Direct Memory Access) moves data directly between NIC and memory without CPU
Techniques: memory-mapped buffers, sendfile(), splice(), io_uring
Dramatically reduces CPU usage and latency
Increases throughput, especially for high-bandwidth applications
Used in high-performance networking stacks (DPDK, io_uring)
Requires OS and hardware support
User space applications need special APIs to leverage
Memory page protection ensures isolation between processes
Enables microsecond-level packet processing latency
Trade-off: More complex implementation; benefits most at high packet rates.

User Space to Kernel Space Packet Route
Definition: Path followed by network packets as they transition between privileged kernel space and unprivileged user space.

Flow - Incoming Packet:

NIC receives packet → DMA into kernel memory (NIC driver)
Interrupt triggers; driver wakes protocol handler
Kernel network stack processes (IP routing, TCP/UDP handlers)
Socket buffer (skb) queues data
Application system call (recv(), read()) triggers context switch
Data copied to user space buffer (traditional) or mapped (zero-copy)
Application processes data in user space
Flow - Outgoing Packet:

Application calls system call (send(), write())
Data copied from user space to kernel buffer (traditional) or mapped
Kernel network stack processes (routing, MAC resolution, fragmentation)
NIC driver queues descriptor
DMA transfers from kernel memory to NIC
NIC transmits packet
Context Switch Cost: Each transition is expensive (~microseconds); minimized in high-performance designs.

Optimization Approaches: DPDK bypasses kernel entirely; io_uring batches operations; zero-copy reduces copying overhead.

Libvirt
Definition: Open-source virtualization management library providing unified interface for multiple hypervisors.

Key Points:

Abstraction layer over hypervisors: KVM, QEMU, Xen, VMware, Hyper-V, etc.
Written in C; remote API support via libvirtd daemon
Configuration using XML domain definitions for VMs
Network configuration: virtual bridges, networks, interfaces
Storage management: volumes, pools, snapshots
Console and serial port access
Live migration support between hosts
Snapshot and checkpoint functionality
Command-line tools: virsh, virt-manager, virt-install
Event notifications and monitoring
Security isolation via AppArmor/SELinux integration
Used by OpenStack, oVirt, and many container/VM orchestration platforms
Stateless design; can manage VMs from any host with credentials
Use Case: Infrastructure management in enterprise and cloud environments.

Node and Pods in Kubernetes
Node
Definition: A physical or virtual machine that runs containerized applications in a Kubernetes cluster.

Key Points:

Smallest deployment unit in Kubernetes cluster
Can be virtual machine, physical server, or cloud instance
Each node runs container runtime (Docker, containerd, CRI-O)
Kubelet agent manages pod lifecycle and reports status
Kube-proxy handles networking and service abstractions
Resource allocation: CPU, memory, storage managed by scheduler
Node status includes Ready/NotReady, MemoryPressure, DiskPressure conditions
Labels enable node selectors for pod scheduling
Taints/tolerations control pod placement restrictions
Heartbeat mechanism (node lease) signals liveness
Each node has unique hostname and IP address
System pods (DNS, network plugins) run on nodes
Node Components: Kubelet, Kube-proxy, Container Runtime, cAdvisor

Pods
Definition: Smallest deployable unit in Kubernetes; one or more tightly coupled containers sharing network namespace.

Key Points:

Usually contains single container (though multiple allowed in sidecar patterns)
Containers share network namespace: same IP address, localhost networking
Containers can share storage volumes
Short-lived, ephemeral entities
Automatically created by controllers (Deployments, StatefulSets, DaemonSets)
Define resource requests (CPU, memory) and limits
Environment variables, ConfigMaps, Secrets passed as configuration
Restart policy: Always, OnFailure, Never
Health checks: liveness probes, readiness probes, startup probes
Pods cannot span multiple nodes
Identified by namespace + name (fully qualified: namespace/podname)
Lifecycle: Pending → Running → Succeeded/Failed
Typical Pattern: One application container + optional sidecar containers (logging, monitoring, networking)

Communication: Pod-to-pod via pod IP (CNI manages addressing); pods discovered via DNS service names.

Veth Pairs (Virtual Ethernet)
What They Are
Veth pairs are virtual network devices that work like a virtual ethernet cable. They always come in pairs, and traffic sent into one end immediately comes out the other end. Think of them as a software-based network pipe.

Primary Use Cases
Container networking: Docker and Kubernetes use veth pairs extensively to connect containers to the host network
Network namespace isolation: Connecting isolated network namespaces to the host or each other
Virtual network topologies: Creating complex network architectures for testing and development
Software-defined networks: Building flexible, programmable network infrastructures
Important Characteristics
Low latency: Everything happens in kernel space, providing efficient communication
Independent configuration: Each end can have its own IP address, MAC address, and MTU settings
Bridge compatibility: Commonly paired with Linux bridges to create complex topologies
Performance: Overhead is minimal but measurable under high load conditions
VLANs (Virtual LANs)
What They Are
VLANs allow you to partition a single physical network into multiple logical networks. Each VLAN operates as if it's on completely separate hardware, providing isolation and segmentation without needing additional physical switches.

Port Types
Access Ports: These ports belong to a single VLAN and send/receive untagged traffic. They are typically used for connecting end devices like computers, printers, or phones.

Trunk Ports: Trunk ports carry traffic for multiple VLANs simultaneously by sending tagged traffic. They are used to connect switches together or to connect switches to routers.

Native VLAN: This is a special designation on trunk ports where untagged traffic is assigned to a specific VLAN. This is an important security consideration, as misconfiguration can lead to vulnerabilities.

Primary Use Cases
Departmental separation: Isolating different departments or functional groups such as HR, Engineering, and Guest networks
Security isolation: Creating security boundaries between different network segments
Traffic type separation: Segregating voice, data, and management traffic
Multi-tenant environments: Providing isolation where multiple tenants share the same physical infrastructure
Important Characteristics
Broadcast domain reduction: VLANs reduce broadcast domains, which improves overall network performance and efficiency
Frame overhead: VLAN tagging adds 4 bytes to each ethernet frame, which may require MTU adjustments in some scenarios
Hardware requirements: Proper VLAN functionality requires VLAN-aware switches
Security considerations: Administrators must watch for VLAN hopping attacks that can occur on misconfigured trunk ports
Inter-VLAN Communication
Devices on different VLANs require a Layer 3 device such as a router or Layer 3 switch to communicate with each other. This process is called inter-VLAN routing and provides a controlled mechanism to allow cross-VLAN traffic based on security policies and requirements.

Combining Veth Pairs and VLANs
Powerful Together
The combination of veth pairs and VLANs enables sophisticated network architectures that leverage the strengths of both technologies. You can create veth pairs to establish connections between containers or namespaces, add VLAN interfaces on top of veth devices for traffic segmentation, and connect everything to VLAN-aware bridges to build complex topologies.
