# Developer Persona

## The Setup  
![](assets/persona-developer-overview.png)

The diagram above illustrates the environment at a high-level. There is an Anthos GKE cluster in which I will deploy my application, with an external Citrix VPX to control ingress traffic into the cluster with support for Web Application Firewall (WAF) protection. This Citrix VPX appliance is managed by external teams and adheres to corporate application delivery standards, but allows me to specify my own WAF configurations.   

As a developer, I am responsible for deploying applications to a Google Anthos platform and ensuring that my application is available and complies to my corporate deployment standards. Network ingress into my application needs to permit and protect access to the application.

## The Why  

Deploying applications into a Citrix Integrated Google Anthos Platform allows me, the developer, to set specific network configurations as simple annotations within my kubernetes manifests. I don't need to learn an additional platform or tool, and this configuration will be applied in accordance with any constraints set out by the platform, network, and security teams. This allows my application to get to market faster with less internal meetings, approvals, or change requests.  

My platform and security teams have requested that I protect my application using Citrix WAF capabilities, but as the developer, I am responsible for ensuring that the WAF configuration is appropriate for my application. Luckily Citrix provides a Kubernetes Custom Resource that I can use to define the right WAF policy without needing to engage with other teams, and still be compliant with my platform and security teams requirements. 


## The How  

---
**NOTE**
In this demonstration, the kubectl binary and local files are used to deploy the application. In production scenarios, other deployment methods would likely be in place, such as a GitOps approach as provided from Google Anthos Configuration Management.  

**Important**
Please note that ADC VPX security features require ADC to be licensed. After ADC VPX is in place, please make sure to follow the steps required to apply your license in one of the various ways that are supported. For simplicity, for this demonstration we are [Using a standalone Citrix ADC VPX license](lab-automation/Licensing.md). For production deployment scenarios you are encouraged to apply different licensing schemes.
- [Licensing overview](https://docs.citrix.com/en-us/citrix-adc/current-release/licensing.html)
- [Citrix ADC pooled capacity](https://docs.citrix.com/en-us/citrix-application-delivery-management-software/current-release/license-server/adc-pooled-capacity.html)

---

After the application and all Citrix components have been deployed on the GKE cluster I will make some changes on the WAF resource and commit the changes to my Git Repository. ACM will make sure that my Infrastructure as Code changes will be resembled to actual Runtime configurations This will outline the control I have over the ingress WAF protection into my application, without the need to engage with the network or security team to make Application Delivery Controller configuration changes. 

- First clone the git repository that the automation created, then deploy the application **Note that you will need to replace the <github-org> and <repo> tags according to your deployment of this lab** 
  ```shell
  sh-5.1$ git clone git@github.com:<github-org>/<repo>.git
  Cloning into '<repo>'...
  remote: Enumerating objects: 301, done.
  remote: Counting objects: 100% (301/301), done.
  remote: Compressing objects: 100% (283/283), done.
  remote: Total 301 (delta 102), reused 0 (delta 0), pack-reused 0
  Receiving objects: 100% (301/301), 33.28 KiB | 2.38 MiB/s, done.
  Resolving deltas: 100% (102/102), done.
  sh-5.1$ cd <repo>/acm/namespaces/demoapp/

  sh-5.1$ cat waf_basic.yaml
  apiVersion: citrix.com/v1
  kind: waf
  metadata:
      name: wafbasic
  spec:
      servicenames:
          - frontend
      security_checks:
          common:
            allow_url: "on"
            block_url: "on"
            buffer_overflow: "on"
            multiple_headers:
              action: ["block", "log"]
          html:
            cross_site_scripting: "on"
            field_format: "on"
            sql_injection: "off"
            fileupload_type: "on"
          json:
            dos: "on"
            sql_injection: "on"
            cross_site_scripting: "on"
          xml:
            dos: "on"
            wsi: "on"
            attachment: "on"
            format: "on"
      relaxations:
          common:
            allow_url:
              urls:
                  - "^[^?]+[.](html?|shtml|js|gif|jpg|jpeg|png|swf|pif|pdf|css|csv)$"
                  - "^[^?]+[.](cgi|aspx?|jsp|php|pl)([?].*)?$"
  ```

- Change sql_injection value from off to on and commit / push changes to Git repo. 
  ```shell
  sh-5.1$ cat waf_basic.yaml
  apiVersion: citrix.com/v1
  kind: waf
  metadata:
      name: wafbasic
  spec:
      servicenames:
          - frontend
      security_checks:
          common:
            allow_url: "on"
            block_url: "on"
            buffer_overflow: "on"
            multiple_headers:
              action: ["block", "log"]
          html:
            cross_site_scripting: "on"
            field_format: "on"
            sql_injection: "on"
            fileupload_type: "on"
          json:
            dos: "on"
            sql_injection: "on"
            cross_site_scripting: "on"
          xml:
            dos: "on"
            wsi: "on"
            attachment: "on"
            format: "on"
      relaxations:
          common:
            allow_url:
              urls:
                  - "^[^?]+[.](html?|shtml|js|gif|jpg|jpeg|png|swf|pif|pdf|css|csv)$"
                  - "^[^?]+[.](cgi|aspx?|jsp|php|pl)([?].*)?$"
  ```
---

Network and security teams can view my configuration from the Citrix ADC including:

- Citrix WAF Policies 
  ![](assets/waf_basic_policy.png)
- Citrix WAF Profiles
  ![](assets/waf_profiles.png)
- Citrix WAF Policy Bindings to Virtual Load Balancing Server 
  ![](assets/waf_basic_policy_binding.png)  
- Citrix WAF Policy Binding Details
  ![](assets/waf_basic_policy_binding_details.png)



To see more configuration options, review the [waf crd examples](https://developer-docs.citrix.com/projects/citrix-k8s-ingress-controller/en/latest/crds/waf/) documentation. 



## Summary  

As a developer, my primary concern is to quickly and securely release my cloud-native application with the pace of my development team and without delays from external teams. Using the Citrix Ingress Controller  and WAF CRDs in a Google Anthos platform allows my team to achieve this goal. 
- Network, Security, and Platform teams can configure sensible defaults and constraints automatically without needing my involvement
- Configurable items specific to my application, **including WAF security**, are delegated to my team in a self-service manner
- Visibility of my workloads are present in the northbound network infrastructure to provide better monitoring and alerting across teams
- I can collaborate in network and security troubleshooting with my network engineers with a shared context and understanding of my workloads

