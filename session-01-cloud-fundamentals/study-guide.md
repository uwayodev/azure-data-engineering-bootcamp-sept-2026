# Session 1: Cloud Fundamentals (Study guide)

**What you will learn:** You will understand what cloud computing is, its main benefits and drawbacks, and how it compares with on-premises. You will also create your free Azure account, ready for the hands-on sessions.

**Before you start:** A laptop or desktop with Google Chrome, your phone for a verification code, and an email address you will keep using.

## 1. What is cloud computing

### In one sentence

Cloud computing means using computing resources (servers, storage, databases, networking, software and analytics) over the internet, on demand, and paying only for what you use.

### An easy analogy

Think of electricity. You do not build your own power station; you plug in and pay for what you consume. The cloud works the same way for computing power and storage.

### Key characteristics

- **On-demand self-service:** you create resources yourself, when you need them.
- **Broad network access:** you reach them over the internet from anywhere.
- **Resource pooling:** the provider shares its hardware across many customers.
- **Rapid elasticity:** resources can grow and shrink quickly.
- **Measured service:** usage is tracked, and you pay for what you use.

### Service models

| Model | What you get | You manage | Azure examples |
| --- | --- | --- | --- |
| **IaaS** | Virtual machines, storage and networking | Operating system, runtime, apps, data | Azure Virtual Machines |
| **PaaS** | A managed platform for apps and data workloads | Your apps and data | Azure SQL Database, Azure Data Factory, Azure Databricks |
| **SaaS** | Ready-to-use software | Your data, users and settings | Microsoft 365, Power BI |

### Deployment models

- **Public cloud:** services run in the provider’s datacenters and are shared across customers (for example Azure).
- **Private cloud:** cloud resources used by a single organisation, often in its own datacenter.
- **Hybrid cloud:** a mix of on-premises and public cloud that work together.

> **Shared responsibility**
>
> The provider is always responsible for the physical datacenters, hardware and core network. You are always responsible for your data, your identities and how you configure your resources. The more managed the service (IaaS to PaaS to SaaS), the more the provider handles for you.

### Link to data engineering

Everything we build in this bootcamp is a cloud service: storing data (ADLS Gen2), moving it (Azure Data Factory), and processing it (Azure Databricks).

**Think about it:** Which cloud services do you already use every day? (For example email, file storage, streaming or mobile apps.)

## 2. Cloud benefits

A short overview before going deeper. The main benefits organisations look for:

- **Cost efficiency:** no large upfront hardware purchase; pay for what you use.
- **Scalability and elasticity:** add or remove capacity as demand changes.
- **Global reach:** deploy close to users in many parts of the world.
- **Agility:** create resources in minutes and try ideas quickly.
- **Reliability:** built-in redundancy, backup and disaster recovery options.
- **Security and compliance:** providers invest heavily in protection and certifications, though you still share responsibility.
- **Managed services:** less time maintaining servers, more time on data and applications.
- **Access to innovation:** analytics, AI and machine learning services ready to use.

Next we look at five of these in depth: scalability, autoscaling, geo distribution, agility and the pay-as-you-go model.

## 3. Scalability

**Scalability** is the ability of a system to handle more (or less) work by adding (or removing) resources.

|   | Scale up (vertical) | Scale out (horizontal) |
| --- | --- | --- |
| **What changes** | A bigger machine: more CPU and memory | More machines sharing the work |
| **Strength** | Simple; often no application changes | Scales much further; can survive a machine failure |
| **Limit** | Hardware ceiling; usually needs a restart | Application must be designed to spread work |
| **Example** | Resize a virtual machine to a larger size | Add more worker nodes to a Spark cluster |

- Example: an online shop needs far more capacity in a sales week than in a quiet month.
- Data grows over time. Cloud storage such as ADLS Gen2 is built to hold very large volumes of data without you buying disks.
- Cloud scalability is on demand; on-premises you must buy and install hardware first.

**Think about it:** When might a system you know suddenly need to handle many more users or much more data?

## 4. Autoscaling

**Autoscaling** means the platform adds or removes resources automatically based on demand, so you do not have to do it by hand.

### How it works

- **Rules** watch a metric (for example CPU usage) or follow a schedule (for example busy weekday mornings).
- **Scale out** when demand rises, and **scale in** when it drops.
- **Minimum and maximum limits** keep performance acceptable and costs under control.

### Why it matters

- With fixed capacity you either waste money when it is quiet or run out when it is busy. Autoscaling follows the demand curve.
- **Scalability vs elasticity:** scalability is the ability to grow; elasticity is growing and shrinking automatically as needed.

### Examples

- Azure Monitor autoscale can adjust services such as App Service and virtual machine scale sets.
- Azure Databricks compute can autoscale between a minimum and a maximum number of workers, and can auto-terminate when idle.

> **Watch out**
>
> Always set a maximum. An autoscaling rule with no upper limit can raise your bill quickly.

**Think about it:** Where would autoscaling help a service you know: a school portal, a mobile-money app, a shop?

## 5. Geo distribution

Cloud providers run datacenters all over the world. **Geo distribution** means placing your data and applications in the locations that suit your users, your resilience needs and your legal requirements.

### Building blocks

- **Region:** a geographic area containing one or more datacenters.
- **Availability zone:** a physically separate location within a region, with its own power, cooling and networking. Not every region offers zones.
- **Region pair:** two regions in the same geography that can back each other up.

### Why it matters

- **Low latency:** put data and apps close to your users.
- **Resilience:** copy data across zones or regions so one failure does not stop you.
- **Data residency:** keep data in a chosen country or geography to meet rules and policies.

### Storage redundancy in Azure (preview)

| Option | What it does |
| --- | --- |
| **LRS** | Keeps copies within a single datacenter |
| **ZRS** | Keeps copies across availability zones in one region |
| **GRS** | Also copies data to a secondary region far away |

- For users in East Africa, nearby Azure regions include South Africa North. Check the current Azure region list, because Microsoft adds regions over time.
- Not every service or price is the same in every region, so check before you choose.

**Think about it:** If your users are in Rwanda, what would you weigh when picking a region: distance, price, or legal rules?

## 6. Agility

**Agility** is how quickly you can turn an idea into something running, and change it when you learn something new.

|   | On-premises | Cloud |
| --- | --- | --- |
| **Getting a server** | Order, ship, install, configure: weeks to months | A few clicks or a script: minutes |
| **Trying an idea** | Costly and slow, so people hesitate | Cheap to try, easy to delete |
| **Changing course** | Hardware is already bought | Resize or remove resources any time |

- Faster experiments mean you can fail fast, learn fast and reach the market sooner.
- Infrastructure can be created from code and repeated reliably (infrastructure as code).
- In Session 2 we will create a storage account and other services in a few minutes each.

**Think about it:** What would you build or test if getting a server took minutes and cost almost nothing?

## 7. Pay-as-you-go model

With **pay-as-you-go** you pay for the resources you actually use, instead of buying capacity up front.

|   | Traditional (CapEx) | Cloud (OpEx) |
| --- | --- | --- |
| **Payment** | Large upfront purchase of hardware | Regular bills based on usage |
| **Capacity** | Guessed in advance; often idle | Matched to actual demand |
| **Risk** | Money is spent before value is proven | Start small and grow as needed |

### How you are billed

- **Compute:** by the time it runs (for example per second, minute or hour, depending on the service).
- **Storage:** by the amount stored over time, and sometimes by how it is accessed.
- **Data transfer:** moving data out of the cloud can carry charges.

### Ways to keep costs under control

- Use the Azure pricing calculator to estimate costs before you build.
- Set budgets and alerts in Cost Management.
- Right-size resources, use autoscaling, and auto-terminate compute when idle.
- Delete what you no longer use. Storage keeps costing money even when compute is off.
- For steady workloads, providers offer discounts if you commit to a period of use.

> **For this bootcamp**
>
> Stop or auto-terminate compute after each session, and delete the resource group when the bootcamp ends. We set a budget alert in Session 2.

**Think about it:** Have you ever been surprised by a bill (phone, electricity, cloud)? What caused it?

## 8. Cons of cloud

The cloud is not perfect. A balanced view builds trust and better decisions.

| Challenge | Why it matters | How to manage it |
| --- | --- | --- |
| **Cost creep** | Pay-as-you-go can grow quickly if resources are left running | Budgets and alerts; stop and delete unused resources; review bills |
| **Internet dependence** | No connection means no access to cloud services | Plan for outages; choose nearby regions; keep local backups where needed |
| **Vendor lock-in** | Moving between providers can be hard and costly | Use open formats (for example Parquet and Delta) and portable skills |
| **Less control** | You do not control the underlying hardware | Understand the shared responsibility model; choose the right service level |
| **Security and compliance** | Misconfiguration is your responsibility | Least-privilege access, strong identity, follow policies and regulations |
| **Data residency and transfer fees** | Rules may limit where data can live; moving data out can cost money | Choose regions carefully; plan data movement |
| **Provider outages** | Rare, but large providers do have incidents | Use zones or regions for critical workloads; have a recovery plan |
| **Learning curve** | Many new services and terms | Learn step by step with hands-on practice, like this bootcamp |

*Billing may be in a foreign currency, so exchange-rate changes can affect budgets.*

**Think about it:** Which of these worries you most for your own work or business?

## 9. On-prem vs cloud

|   | On-premises | Cloud |
| --- | --- | --- |
| **Cost model** | Upfront purchase (CapEx) | Pay as you go (OpEx) |
| **Setup time** | Weeks to months | Minutes to hours |
| **Scaling** | Limited by the hardware you bought | On demand, up or down |
| **Maintenance** | Your team looks after everything | Provider looks after the infrastructure |
| **Control** | Full control | Shared control |
| **Security** | You are fully responsible | Shared responsibility |
| **Reliability** | You design and build redundancy | Redundancy options built in |
| **Best for** | Stable workloads, strict local requirements | Variable or fast-growing workloads |

- **Hybrid** is common: keep some systems on-premises and use the cloud for others, connected together.
- Neither is always right. The answer depends on cost, data rules, skills, connectivity and how fast the workload changes.

### Quick scenarios to discuss

- A bank with strict rules on where customer data can be stored.
- A startup whose traffic could double next month, or stay flat.
- A small business with a stable in-house system that rarely changes.

**Think about it:** Which option would you choose for each of these, and why?

## 10. Create your free Azure account

You do not create any other resources in this session; that starts in Session 2.

### Before you start

- Use Google Chrome on a laptop or desktop.
- Have a phone that can receive a verification code.
- A payment or debit card may be needed for identity verification.
- Use an email address you will keep using for the bootcamp. A personal Microsoft account is simplest; work or school accounts may be restricted by an organisation.

### Step by step

1. Open the Azure free account page in Chrome and select **Start free**.
2. Sign in with your Microsoft account, or create one.
3. Enter your country or region, your name and your phone number, then verify the phone number.
4. Verify your identity with a payment card. Read the terms shown at sign-up before continuing.
5. Accept the agreement and finish sign-up.
6. Open **portal.azure.com** and check that you can see your subscription.

### If something goes wrong

- **Card verification fails:** try another card, and check that it supports online payments.
- **Sign-up page errors or loops:** try a private browsing window, then another browser.
- **Cannot create an account at all:** Databricks Free Edition can cover the Databricks parts of the course.

## Wrap-up and next steps

### Prepare before Session 2

- Confirm you can still sign in to the Azure portal.
- Join from a laptop or desktop with Google Chrome.
- Have a look around the Azure portal, but do not create resources yet.
- Post any blockers in the group so they can be fixed before Session 2.

## Practice: quiz and assignment

### Quiz

1. Which best describes cloud computing?
   - A. Buying servers and running them in your own datacenter
   - B. Using computing resources over the internet, on demand, and paying for what you use
   - C. Storing files only on your laptop
   - D. A programming language
2. Which service model gives you a managed platform where you manage mainly your applications and data?
   - A. IaaS
   - B. PaaS
   - C. On-premises
   - D. None of these
3. What is the difference between scaling up and scaling out?
4. Why should you set a maximum on an autoscaling rule?
5. Under the shared responsibility model, who is always responsible for your data, identities and how you configure your resources?
   - A. The cloud provider
   - B. You
   - C. Nobody
   - D. Only your internet provider
6. Name two cons of the cloud and one way to manage each.

<details>
<summary>Show the answers</summary>

1. B. Cloud computing means using computing resources over the internet, on demand, and paying for what you use.
2. B. PaaS (for example Azure Data Factory and Azure Databricks). You manage your apps and data; the provider manages the platform.
3. Scaling up means using a bigger machine (more CPU and memory). Scaling out means using more machines that share the work.
4. An autoscaling rule with no upper limit can add resources without stopping and raise your bill quickly. A maximum keeps cost under control.
5. B. You. The provider secures the physical datacenters and hardware; you secure your data, identities and configuration.
6. For example: cost creep (set budgets and alerts, delete unused resources) and vendor lock-in (use open formats and portable skills). Other cons: internet dependence, less control, security and compliance responsibilities, data residency and transfer fees.

</details>

### Assignment

1. Create your free Azure account if you have not yet, then sign in to portal.azure.com.
2. Open **Subscriptions** and note the name of your subscription.
3. Do **not** create any resources yet. That starts in Session 2.
4. Write 5 to 6 sentences explaining to a friend what the cloud is and two of its benefits, using an example from daily life.
5. List two differences between on-premises and cloud for a small business you know.
6. Take a screenshot of the portal home page. Hide any personal details.

**What to submit:** Your screenshot and your short explanation, shared as your instructor asks. Share it the way your instructor asks (for example in the course group), and never include passwords, keys or card details.

## Key terms

| Term | Meaning |
| --- | --- |
| Cloud computing | Using computing resources over the internet, on demand, and paying for what you use |
| IaaS / PaaS / SaaS | Infrastructure, platform or software delivered as a service |
| Public / private / hybrid cloud | Shared provider cloud / single-organisation cloud / a mix with on-premises |
| Shared responsibility | The split of security duties between the provider and the customer |
| Scalability | Ability to handle more or less work by changing resources |
| Scale up / scale out | Use a bigger machine / use more machines |
| Autoscaling | Automatic adding and removing of resources based on demand |
| Elasticity | Growing and shrinking capacity automatically as demand changes |
| Region / availability zone | A geographic area of datacenters / a separate location inside a region |
| Data residency | Keeping data within a chosen country or geography |
| Agility | How quickly you can create, change and remove resources |
| CapEx / OpEx | Upfront capital spending / ongoing operating spending |
| Vendor lock-in | Difficulty moving away from one provider |
