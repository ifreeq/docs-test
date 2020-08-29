# The Technical Data Guidance

## EN

The  following key words about network are the technologies that I had study and used. And now I'm studying the fourth generation of web application technologies and the Firebase dynamic database, after I  fully understand  and can use these technologies, I will sort out them again

### The keywords for third generation web platform

#### **Compute Engine:** Google Cloud based on United States\(us-west2-a\)

> Supplement: GCP is for formal plan. After I test Alicloud, Tencent Cloud, Amazon Cloud, I found GCP is the best choice.

#### **IP:** Google Anycast IP + Premium Host IP

| Category | **Premium** | Standard |
| :--- | :--- | :--- |
| Network | High performance routing | Lower performance network |
| Network Services | Network services such as Cloud Load Balancing are global \(single VIP for backends in multiple regions\) | Network services such as Cloud Load Balancing are regional \(one VIP per region\) |
| Service Level | High performance and reliability, Low latency, Global SLA | Performance and availability comparable to other public cloud providers \(lower than premium\), no Global SLA |
| Use Case | Performance, reliability, global footprint and user experience are your main considerations | Cost is your main consideration, and you’re willing to trade-off some network performance |

#### **DNS, CDN, Network Load Balancer**

**DNS**

![Google Cloud DNS](../../.gitbook/assets/image%20%288%29.png)

**CDN & Network Load Balancer**

![Traffic Data](../../.gitbook/assets/image%20%287%29.png)

![The Load Balancer](../../.gitbook/assets/image%20%284%29.png)

#### **Picture Bed**

The Picture Bed is based on Google Storage\(US area\) and also use Google CDN with custom domain.

#### SSL

GTS CA 1D2

![Google Trust Service](../../.gitbook/assets/image%20%285%29.png)

#### Web Serve

Ngingx & Tengine

### The Color Guidance For  Design

#### Theme Color

* 蓝色\#4285F4
* 绿色\#0F9D58
* 黄色\#F4B400
* 红色\#DB4437

#### The Font Color

* 4d4d4d
* 1d1d1f
* 424242

#### The Background Color

* f5f5f7
* F7F9FA

#### The Divider Color

* D2D2D7

