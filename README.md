## Background
Security is a highly dynamic topic with ever changing threats and priorities. Newsworthy topics ranging from fortune 500 companies like [Garmin](https://www.wired.com/story/garmin-ransomware-hack-warning) paying $10 million in ransom for ransomware attacks to supply chain attacks such as [Lapsus$](https://www.theverge.com/2022/4/20/23034360/okta-lapsus-hack-investigation-breach-25-minutes) are ever-present. 

Security is becoming harder for engienering teams as the velocity of service deployments is accelerating. The [Synopsis 2022 Open Source Security Risk Analysis Report](https://www.synopsys.com/content/dam/synopsys/sig-assets/reports/rep-ossra-2022.pdf) revealed that 78% of audited code bases was open source, and within those codebases, 81% contained at least one vulnerabiltiy, which creates risk if left unpatched. With "shifting right", the industry has doubled down on incorporating security validations into each step of the build -> from the local dev environemnt -> to linters commit and pull request review -> to ci/cd checks during the deployment process, incorporating check into nominal deployment process are vital to identify security defects before they hit production.

Your company CTO is worried about what your engineering team is doing to harden and monitor the company's new microservices against malicious threat actors and payloads. You’ve completed the exercies in the course and have a baseline understanding of how to approach this. In response to the CTOs concerns, students will threat model and build a hardened  microservices stack based on what they learned from the exercises.

## Goal 
You will be presented with the challenge to build a secure microservice stack, threat modeling and hardening the container image, run-time environment and application itself. For purposes of the project, you will be instructed to use a secure base opensuse image, covering considerations for the importance of using trustworthy base images and verifing the baselein. You will be provided with instructions to build, harden, ship and run an environment analogous to the company's new microservice, simplified for project purposes. In the project you will define and build a new environment from the ground-up. 

In a real-world scenario, you may have an existing envrionment that needs to be hardened or it may decided to re-build parts or all net-new, regardless, the tools and techniques in the project are directly applicable. The beauty of microservices vs a monolith architecture is that all core components (image, container, run-time, application) are abstracted allowing for isolation boundaries and iterative development. In the real-world, you could chose to harden and redeploy all base-images as one project phase and tackle docker container security, kubernetes hardening and the software composition anaylsis, as individual project phases. 

The best approach is to incorporate these requirements and security hardening into the build and deploy processes. In an enterprise setting, much of this can be enforced with linters on commit and PR review and security units test via CI/CD prior to deployment. Hardening the base-image and incorporating security checks into the CI/CD is beyond the scope of this project and course, however please reference the [additional considerations](https://github.com/udacity/nd064-c3-Microservices-Security-project-starter/tree/master/starter#additional-considerations) section for more on this. 

For the project, once the microservice stack is hardened and provisioned, we will configure [sysdig Falco](https://github.com/falcosecurity/falco) to perform run-time monitoring on the node, sending logs to a Grafana node for visualization. To demonstrate to the CTO that the company can respond to a real security event, you will then simulate a [tabletop cyber exercise](https://www.tripwire.com/state-of-security/security-data-protection/everything-you-need-to-know-about-cyber-crisis-tabletop-exercises/) by running a script to introduce an unknown binary from the starter code that will intentionally disrupt the environment.  

No stress, you have tools, security engineering and security incident response knowledge to respond :) Your goal will be to use Falco events visualized in Grafana to determine what the unknown binary is, contain and remediate the environment, write a post-mortem incident response report and present it to the CTO. There will be a few hidden easter eggs, see if you can find them for extra credit! 

## Project Instructions

### The _/starter_ directory contains:
```bash
├── Dockerfile 
├── LICENSE
├── Vagrantfile
├── cluster.yml                     # to provision a Rancher RKE 1-node cluster
├── reference_hardened_cluster.yml  # is a reference hardened cluster.yml to guide you. You cannot use the reference_hardened_cluster.yml file as-is to startup the cluster, it needs to be revised
├── docs                            # contains reference PDFs. you will use these for reference
│   ├── CIS_Docker_Benchmark_v1.2.0.pdf
│   ├── CIS_Kubernetes_Benchmark_v1.6.0.pdf
│   └── Rancher_Benchmark_Assessment.pdf
├── incident_response               # directory contains an incident_response.txt template responding to an incident
│   └── incident_response.txt
├── scripts                         # directory contains a payload.sh that will be used later in the project
│   └── payload.sh
├── security_architecture           # directory contains am example security architecture diagram.
└── threat_modeling                 # directory contains a threat_modeling_template.txt template, use it for STRIDE threat modeling
    └── threat_modeling_template.txt 
```

### The _/vuln_app_ is a popular vulnerable web application:
```bash
└── vuln_app
    ├── Dockerfile.app      # defines that this is a dockerized application
    ├── Dockerfile.db       # indicates that there is a Dockerized database
    ├── LICENSE
    ├── README.rst
    ├── config              # contains configurations to startup the application
    ├── docker-compose.yml  # Run this to bring up the application
    ├── migrations          # folder contains migration scripts
    ├── recreate.sh         # allows us to recreate the database
    ├── requirements.txt
    ├── run.py              # the primary Python app startup directory
    └── sqli                # is the configuration directory for the SQLite database that's run as part of the application
```
To run this **vuln_app** follow these steps:
1. Run the Flask program by using the docker-compose up command.
2. The application should be running on port 8000. You can access it by querying the http://localhost:8080endpoint.

### Step 1: Threat Model the Microservices Environment
Define a security architecture for your environment and identify attack surfaces. The environment consists of the following:
- Single OpenSUSE Linux virtual machine (host)
- An RKE cluster deploy to the Linux virtual machine node cluster (cluster)
- The cluster runs an intentionally vulnerable python application (service)
- Docker is running on the host node to manage containers (container daemon)

Create a diagram of the environment you are about to implement.
- Your diagram should minimally abstract the openSUSE host, RKE cluster, Python service, and Docker container daemon you will deploy.
- You should identify service and security boundaries with lines and data flow with arrows. Refer to the example on the page Monolithic vs Microservices Security Boundaries in the lesson Threat Modeling with STRIDE in the classroom.

submissions/security_architecture_design.png

Using the STRIDE threat modeling methodology and the threat_modeling_template.txt in the /starter/threat_modeling directory of the project repo, document 5 concrete attack surface areas for the Docker environment and a total of 5 concrete attack surfaces for the Kubernetes control plane, etcd, and worker. In your explanation, associate each attack surface area to at least one pillar of the STRIDE model, which includes Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege. There can be multiple attack surface areas associated with one pillar, but the attack surfaces have to be distinct.

submissions/threat_modeling_template.txt

### Step 2: Harden the Microservices Environment
#### Create a Hardened Docker Environment
1. Identify weaknesses:
Using the starter Dockerfile in the starter repo and an openSUSE base image, create a hardened Docker environment with Docker-bench:
- Run Docker-bench for the first time. Take screenshots of the result summary and all failed findings, 

submissions/ suse_docker_environment_out_of_box.png

- Using the CIS_Docker_Benchmark_v1.2.0.pdf from the starter repo, review the findings from running the docker-bench.
- From the failed findings, select and document 3 failed findings from the Docker-bench results that you want to harden. These 3 findings should confirm 3 out of the 5 attack surfaces you identified for Docker in Step 1. At least 1 of the 3 findings should be different from the ones mentioned in the exercise (i.e. 5.10, 5.14, and 5.22). Please be mindful and don't submit the same surfaces, the goal here is learn and understand a variety of surfaces.
- Document each of the 3 findings you want to harden to the existing

submissions/threat_modeling_template.txt

2. Harden Weaknesses:
- Harden the three Docker weaknesses you identified.
- Re-run Docker-bench to verify that the weaknesses you hardened have been addressed.

submissions/suse_docker_environment_hardened.png

#### Create a Hardened Kubernetes Environment
- Deploy an RKE cluster using the Vagrantfile.
- Run Kube-bench for the first time. Take screenshots of the result summary and all failed findings

submissions/kube_cluster_out_of_box.png

- Apply [baseline hardening steps](https://ranchermanager.docs.rancher.com/v2.0-v2.4/reference-guides/rancher-security/rancher-v2.4-hardening-guides/hardening-guide-with-cis-v1.5-benchmark) to the cluster.
- Re-run Kube-bench to verify the cluster has been hardened via baseline hardening. Take screenshots of the result summary and all failed findings

submissions/kube_cluster_hardened.png

- The most important aspect of hardening is making sure the hardening does not negatively affect system stability. In the real world all security related changes should be tested on production like systems. The last thing you want as a security engineer is for your hardening to lead to an outage of the Kubernetes cluster. Write at least 200 words describing a Kubernetes-specific test plan based on what you learned from the course. The test plan does not need to address specific hardening steps. Answer these two questions in your test plan:
  - How will you test the changes?
  - How will you ensure the changes don't negatively affect your cluster?

submissions/kube_hardening_test_plan.txt

### Step 3: Harden and Deploy the Flask App
Here we will focus on hardening and deploying the vulnerable Python Flask app by performing software introspection to identify and remediate vulnerable libraries and code.
- The application has intentional security flaws in the code that you need to identify and remediate using your knowledge. There are four known appsec vulnerabilities. You will need to minimally remediate the Cross-Site Scripting (XSS) vulnerability in the code and redeploy the app.
  - Fix minimally the Cross-Site Scripting (XSS) vulnerability in the app.py file located in dvpwa/sqli/app.py. 
  
  submissions/app.py
  
  - You will get extra points if you can research and remediate any other vulnerabilities in the code, there are three more! Call out the specific line(s) of code you've changed in the app.py file or any other relevant file to remediate the vulnerabilities.
- Configure and run Grype to identify vulnerabilities in the libraries and remediate them.
  - Run a Grype scan in the terminal for the first time. Take a screenshot of all Grype findings
  
  submissions/grype_app_out_of_box.png

  - Research vulnerable libraries on the NVD website and remediate them.
  - Re-run Grype until all vulnerable libraries are remediated. Take a screenshot of the Grype output showing 0 findings 
  
  submissions/grype_app_hardended.png

- With the Python Flask app hardened, redeploy the app by using docker compose up and access the application on the localhost port 8080 (127.0.0.1:8080).

### Step 4: Implement Runtime Monitoring and Grafana
Implementing Grafana to visualize run-time security alerts generated via Sysdig Falco.

1. Deploy Helm, Falco SUSE specific headers, Falco SUSE specific drivers +checksum, Falco daemon, and falco-exporter to move love from the pod to grafana. This is a key step, make sure you install the SUSE-specific kernel headers prepared by the Falco team in order to intercept syscalls on the SUSE operating system. Non-SUSE headers and drivers will not work.
  - GETTING HELP! As falco header and driver installation can be very nuanced, please carefully reference the classroom content. If you get stuck, carefully re-read the classroom content and reference the classroom "Falco kmod and eBPF troubleshooting" page, where we provide substantive troubleshooting steps. If you are blocked, please reach out to the #falco channel in the Kubernetes Slack to ask for advice from the community or ask a Udacity mentor. The falco community is very passionate and is often willing to help follow engineers. Do not give up!
  - Take a screenshot of the Falco and falco-exporter pods running. 

submissions/kube_pods_screenshot.png

  - Provide evidence that Falco is generating security events by reading the content of a sensitive file. Take a screenshot of the warning message(s) from Falco pod logs or from the falco-exporter metrics page

submissions/falco_alert_screenshot.png

2. Next, configure Falco to send security events to Grafana:
  - Configure the Prometheus Operator and Grafana.
  - Import the Falco panel for Grafana. At this point, you should have Grafana running with Falco logs flowing. If the Falco events are not showing up on Grafana, you should repeat steps 1-3 above to generate Falco events.
  - Take a screenshot of the Falco Grafana panel showing the Falco security event.
  
  submissions/falco_grafana_screenshot.png

  - GETTING HELP! If events are not being generated, its most likely that either 1.

3. Optional Challenge:
  - Falco also allows custom rules to be defined. Following the syntax in /etc/falco/falco_rules.yaml, create at least one new rule in the local rule base located at /etc/falco/falco_rules.local.yaml.

### Step 5: Incident Response
Introducing a suspicious command onto the Kubernetes cluster simulating a security incident. A payload is a script or file that delivers a malicious action such as running malware:
- Run the payload.sh to introduce a suspicious command intentionally. The kubectl run command in the payload.sh script will bootstrap containers instantiated with legacy Docker images, such as servethehome/monero_cpu_moneropool, on your cluster.
- We use these as a canonical examples as they are reliable and clearly illustrate falco in action in a controlled learning environment. Those Docker images will run crypto mining software and have multiple security issues, such as not using the secure communication channels and not having a software bill of materials used in the image.
- We have intentionally chosen these Docker images to simulate a "controlled" security incident. Depending on the sophisication and care, attacks may use similar techniques. Executing such suspicious workloads in a controlled environment poses a small security risk, particularly if your system is not patched. To be ultra cautious, make sure your host system is patched before you run the crypto demo to reduce the risk. In the real world, patching is a vital control, always make sure your host systems are patched, and remember that attackers do not ask for permission to attack your system.
- Using the template in the repo, write an incident response report to the CTO to describe what happened. Make sure to be thoughtful and precise as you are writing to an executive. Write at least two sentences for each of the questions in Questions 2-6. 

submissions/incident_response_report.txt

### 3 challenges: Standout Suggestions:
- Research and remediate any vulnerabilities other than XSS in the code of vuln_app. There are 3 others documented, you will need to research how to remediate one or more of them. Call out which line(s) of code you've changed in the app.py file or any other relevant file to remediate the vulnerabilities.
- Create at least one new custom Falco rule in the local rule base located at /etc/falco/falco_rules.local.yaml.
- Test the rule you created in Step 2 by creating a payload (such as shell script with the action the rule monitors for) to trigger the rule.