# The Technical Data Guidance

## EN

The  following key words about network are the technologies that I had study and used. And now I'm studying the fourth generation of web application technologies and the Firebase dynamic database, after I  fully understand  and can use these technologies, I will sort out them again

### The keywords for third generation web platform

**Compute Engine:** Google Cloud based on United States\(us-west2-a	\)

> Supplement: GCP is for formal plan. After I test Alicloud, Tencent Cloud, Amazon Cloud, I found GCP is the best choice.

**IP:** Google Anycast IP + Premium Host IP

| Category | **Premium** | Standard |
| :--- | :--- | :--- |
| Network | High performance routing | Lower performance network |
| Network Services | Network services such as Cloud Load Balancing are global \(single VIP for backends in multiple regions\) | Network services such as Cloud Load Balancing are regional \(one VIP per region\) |
| Service Level | High performance and reliability, Low latency, Global SLA | Performance and availability comparable to other public cloud providers \(lower than premium\), no Global SLA |
| Use Case | Performance, reliability, global footprint and user experience are your main considerations | Cost is your main consideration, and you’re willing to trade-off some network performance |

**DNS, CDN, Network Load Balancer**

**DNS**

![Google Cloud DNS](../../.gitbook/assets/image%20%285%29.png)

**CDN & Network Load Balancer**

![Traffic Data](../../.gitbook/assets/image%20%286%29.png)

![The Load Balancer](../../.gitbook/assets/image%20%287%29.png)



