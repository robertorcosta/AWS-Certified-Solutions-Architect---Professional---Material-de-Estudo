# AWS Certified Solutions Architect Professional (SAP-C02) - Material de Estudo

## Informações do Exame
- Código: SAP-C02
- Custo: $300 USD
- Duração: 180 minutos (3 horas)
- Questões: 75 (65 pontuadas + 10 não pontuadas)
- Aprovação: 750/1000
- Tipo: Múltipla escolha e múltipla resposta
- Pré-requisito recomendado: AWS SAA (Associate) + 2 anos de experiência

## Domínios do Exame
- Domínio 1: Soluções para Complexidade Organizacional (26%)
- Domínio 2: Design de Novas Soluções (29%)
- Domínio 3: Melhoria Contínua para Soluções Existentes (25%)
- Domínio 4: Migração e Modernização de Workloads (20%)

---

## DOMÍNIO 1: Complexidade Organizacional (26%)

### 1.1 Conectividade de Rede

#### AWS Transit Gateway
- Hub central para conectar múltiplas VPCs e redes on-premises
- Suporta peering inter-Region com tráfego criptografado
- Route Tables permitem segmentação de tráfego (ex: isolar prod de dev)
- Até 50 Gbps por attachment, suporta multicast
- Transit VIF do Direct Connect conecta ao TGW
- Cenário na prova: "Empresa com 50+ VPCs precisa de conectividade full-mesh com roteamento centralizado" = Transit Gateway

#### AWS Direct Connect
- Conexão dedicada entre data center e AWS
- Tipos: Dedicated (1/10/100 Gbps) e Hosted (50Mbps a 10Gbps)
- Virtual Interfaces: Private VIF (VPCs), Public VIF (serviços públicos), Transit VIF (TGW)
- LAG para agregar conexões, MACsec para criptografia L2 (10/100 Gbps)
- Resiliência: 2 conexões em locais diferentes + VPN como fallback
- Cenário: "Migração de grandes volumes com latência consistente" = Direct Connect

#### AWS Site-to-Site VPN
- IPsec via internet pública, ~1.25 Gbps por túnel (2 túneis/conexão)
- Accelerated VPN: usa Global Accelerator para melhor performance
- ECMP com Transit Gateway: agregar múltiplas VPNs para mais throughput
- Cenário: "Backup do DX" ou "conexão rápida enquanto DX é provisionado" = VPN

#### VPC Peering vs Transit Gateway vs PrivateLink
- VPC Peering: 1-para-1, NÃO transitivo, sem custo de hora, cross-region/account
- Transit Gateway: Hub-and-spoke, transitivo, $0.05/h por attachment, muitas VPCs
- PrivateLink: expõe serviço específico, Interface Endpoint (ENI + IP privado)
- Gateway Endpoint: S3 e DynamoDB apenas, GRÁTIS, rota na route table
- Interface Endpoint: quase todos os outros serviços, $0.01/h + dados

#### DNS Híbrido (Route 53 Resolver)
- Inbound Endpoint: on-prem resolve nomes na AWS
- Outbound Endpoint: AWS resolve nomes on-prem
- Resolver Rules: encaminhamento condicional (ex: *.corp.local → DNS on-prem)
- Compartilháveis via RAM entre contas

#### Global Accelerator vs CloudFront
- Global Accelerator: Anycast IP (camada 4), TCP/UDP, sem cache, failover instantâneo, 2 IPs estáticos
- CloudFront: CDN (camada 7), HTTP/HTTPS, com cache, DNS CNAME
- GA para: gaming, IoT, VoIP, apps que precisam de IP fixo
- CF para: sites, APIs, streaming

### 1.2 Controles de Segurança

#### IAM Avançado e Cross-Account Access
- AssumeRole: método preferido para cross-account (temporário, auditável)
- Resource-based Policies: S3 bucket policy, KMS, SQS — acesso direto sem assumir role
- Ao assumir uma role, você PERDE suas permissões originais
- Com resource-based policy, você MANTÉM suas permissões originais
- SCPs: limites máximos no nível de Organization (guardrails)
- Permission Boundaries: limite máximo para roles/users dentro de uma conta
- Ordem de avaliação: SCP → Permission Boundary → Identity Policy → Resource Policy → Session Policy
- Deny explícito em QUALQUER nível sempre vence
- Para cross-account: AMBOS os lados precisam permitir (exceto S3 resource-based)

#### IAM Policy Variables e Conditions
- Variables: ${aws:username}, aws:CurrentTime, aws:SourceIp, aws:SecureTransport
- Conditions: StringEquals, IpAddress, Bool (MFA), DateLessThan, Null
- Tag-based: aws:PrincipalTag, iam:ResourceTag
- aws:TagKeys com ForAllValues/ForAnyValue para exigir tags específicas

#### IAM Identity Center (ex-AWS SSO)
- SSO para múltiplas contas AWS + apps SAML 2.0 + EC2 Windows
- Permission Sets: coleção de IAM Policies atribuída por conta
- ABAC: permissões baseadas em atributos (cost center, title, locale)
- Identity sources: built-in, AD, Okta, Azure AD
- Substitui SAML 2.0 Federation como método recomendado

#### Identity Federation
- SAML 2.0: IdP corporativo compatível (ADFS, Okta) → AssumeRoleWithSAML
- Custom Identity Broker: IdP NÃO compatível com SAML → AssumeRole/GetFederationToken
- Web Identity (Cognito): apps mobile/web → AssumeRoleWithWebIdentity
- STS APIs: AssumeRole, AssumeRoleWithSAML, AssumeRoleWithWebIdentity, GetSessionToken (MFA), GetFederationToken

#### AWS Directory Services
- AWS Managed Microsoft AD: AD completo na AWS, trust bidirecional com on-prem, MFA, Multi-AZ, replicação multi-region
- AD Connector: proxy/gateway para AD on-prem, sem cache, sem trust, sem SQL Server, sem seamless join
- Simple AD: básico standalone, sem trust, sem MFA, sem SSO, alternativa barata
- Managed AD + trust com on-prem: para reduzir latência na cloud

#### AWS KMS (Key Management Service)
- Symmetric (AES-256): encrypt/decrypt, usada por todos serviços integrados, envelope encryption
- Asymmetric (RSA/ECC): par público/privado, encrypt fora da AWS
- Customer Managed Key: controle total, rotação 90-2560 dias, Key Policy + CloudTrail
- AWS Managed Key: gerenciada (aws/s3, aws/ebs), rotação obrigatória 1 ano
- AWS Owned Key: interna da AWS, sem visibilidade
- Key Material Origin: KMS (default), External (BYOK, sem rotação auto), Custom Key Store (CloudHSM)
- Multi-Region Keys: mesmo key ID em múltiplas regiões, encrypt/decrypt sem cross-region API call
- Cross-Account: Key Policy permite conta externa → Role na conta externa usa a chave
- Keys são REGIONAIS (exceto multi-region)

#### CloudHSM
- Hardware dedicado (FIPS 140-2 Level 3)
- Você gerencia as chaves (AWS não tem acesso)
- Use quando: compliance requer HSM dedicado, SSL offloading, Oracle TDE
- Integra com KMS via Custom Key Store (mínimo 2 HSMs ativos)

#### CloudTrail
- Auditoria de API calls, habilitado por default
- Management Events: operações em recursos (por padrão)
- Data Events: S3 GetObject/PutObject, Lambda Invoke (NÃO habilitado por padrão)
- Insights Events: detecta atividade incomum (bursts, picos)
- Retenção: 90 dias → S3 + Athena para long-term
- Organization Trail: uma trail captura TODAS as contas-membro
- Log File Integrity: SHA-256 hashing e signing
- Reagir a eventos (mais rápido → mais lento): EventBridge > CloudWatch Logs > S3 delivery (5min)
- Multi-Account Logging: S3 bucket centralizado com bucket policy cross-account
- Alertas: CloudTrail → CW Logs → Metric Filter → Alarm → SNS

#### Serviços de Segurança
- GuardDuty: detecção de ameaças com ML, delegated admin
- Security Hub: visão consolidada de findings + compliance (CIS, PCI DSS)
- AWS Config: avalia conformidade de recursos, Aggregator multi-account/region, remediation via SSM
- IAM Access Analyzer: identifica recursos expostos externamente, zone of trust, policy generation (90 dias CloudTrail)
- Amazon Inspector: scan vulnerabilidades EC2/ECR/Lambda
- Amazon Macie: dados sensíveis em S3 (PII)
- Todos suportam delegated admin para conta de segurança

#### Secrets Manager vs Parameter Store
- Secrets Manager: $0.40/mês, rotação AUTOMÁTICA (Lambda para RDS/Redshift/DocumentDB), KMS obrigatório, cross-account via resource policy, 64KB max
- Parameter Store: grátis (standard 4KB, 10K params), hierarquia (/app/prod/key), SEM rotação nativa (EventBridge+Lambda), KMS opcional
- Se "rotação automática de credenciais de banco" → Secrets Manager
- Se "configurações com hierarquia" → Parameter Store
- Parameter Store pode REFERENCIAR secrets do Secrets Manager via /aws/reference/secretsmanager/

### 1.3 Resiliência e Disaster Recovery

#### Estratégias de DR
- Backup & Restore: horas RTO/RPO, menor custo
- Pilot Light: core mínimo rodando (ex: DB replicado), 10s minutos RTO
- Warm Standby: sistema reduzido ativo, minutos RTO
- Multi-Site / Hot Standby: full active-active, near-zero RTO, mais caro
- Regra: "menor custo" + DR = Backup & Restore. "RTO mínimo" = Multi-Site.

#### Serviços para DR
- Elastic Disaster Recovery (DRS): replicação contínua block-level, failover em minutos, substitui CloudEndure
- Aurora Global Database: RPO < 1s, RTO < 1min, até 5 regiões secundárias
- DynamoDB Global Tables: multi-region active-active, replicação automática
- ElastiCache Global Datastore: Redis cross-region
- S3 Cross-Region Replication: versioning obrigatório, S3 RTC (99.99% em 15min)
- RDS Multi-AZ: failover automático síncrono (30-60s)
- RDS Cross-Region Read Replicas: assíncrono, pode ser promovido

#### Alta Disponibilidade
- Route 53 Routing: Failover (ativo/passivo), Latency (menor latência), Weighted (% tráfego), Geolocation, Multivalue Answer
- Auto Scaling Groups: Target Tracking, Step, Scheduled, Predictive (ML)
- ELB Cross-Zone: distribui uniformemente entre AZs

#### AWS Backup
- Centraliza backups cross-service (EC2, EBS, S3, RDS, Aurora, DynamoDB, EFS, FSx, DocumentDB, Neptune)
- Cross-region e cross-account, PITR, tag-based policies
- Backup Vault Lock (WORM): nem root user pode deletar! Proteção contra ransomware.

#### Fault Injection Simulator (FIS)
- Chaos Engineering gerenciado, templates pré-construídos
- Suporta EC2, ECS, EKS, RDS
- Stop automático se alarm disparar

### 1.4 Multi-Account

#### AWS Organizations
- Hierarquia de OUs, Management Account (não use para workloads)
- SCPs: restringem o que contas PODEM fazer, NÃO concedem permissões
- SCPs NÃO afetam Management Account nem Service-Linked Roles
- Precisam de Allow explícito da root até o alvo na hierarquia
- Consolidated Billing: uma fatura, volume discounts compartilhados
- Reserved Instances são compartilhadas (pode desligar sharing por conta)
- OrganizationAccountAccessRole: full admin em member accounts (auto em novas, manual em convidadas)
- Tag Policies: padronizar tags, validar valores
- Backup Policies: backup plans centralizados (imutáveis nas member accounts)
- AI Services Opt-out: opt-out de ter conteúdo usado por AI/ML da AWS
- Restringir por Região: aws:RequestedRegion em SCP
- Exigir Tags: SCP com aws:TagKeys + ForAllValues

#### AWS Control Tower
- Landing Zone automatizada com best practices
- Guardrails: Preventive (SCPs), Detective (Config Rules), Proactive (CFN hooks)
- Levels: Mandatory (auto), Strongly Recommended (optional), Elective (optional)
- Account Factory: provisiona contas padronizadas via Service Catalog
- Detect & Remediate: Config detecta → SNS → EventBridge → Lambda corrige

#### AWS RAM (Resource Access Manager)
- Compartilha recursos entre contas sem duplicação
- VPC Subnets (cada conta gerencia seus recursos), Transit Gateway, Route 53 Rules, Aurora, CodeBuild, EC2 Dedicated Hosts, Glue Catalog, Network Firewall Policies

### 1.5 Otimização de Custos

#### Modelos de Compra
- On-Demand: sem compromisso, flexibilidade total
- Savings Plans (Compute): até 66%, qualquer instância/região/OS
- Savings Plans (EC2 Instance): até 72%, família + região fixa
- Reserved Instances: até 72%, capacity reservation garantida em AZ
- Spot Instances: até 90%, interruptível com 2min aviso
- Savings Plans = flexibilidade. RI = capacity reservation. Spot = batch/stateless.

#### Ferramentas de Custo
- Cost Explorer: visualização + forecast
- AWS Budgets: alertas + ações automáticas (SCP, stop instances)
- CUR (Cost and Usage Report): granular → S3 → Athena/QuickSight
- Compute Optimizer: rightsizing EC2/EBS/Lambda/ECS
- S3 Storage Lens: análise org-wide (28 métricas grátis, 14 dias)
- Trusted Advisor: economia, segurança, performance, limites

---

## DOMÍNIO 2: Design de Novas Soluções (29%)

### 2.1 Estratégias de Deploy

#### Modelos de Deployment
- All-at-once: atualiza tudo, alto risco, re-deploy para rollback
- Rolling: atualiza em lotes, risco médio
- Blue/Green: ambiente paralelo, switch via DNS/LB, rollback instantâneo
- Canary: % pequeno do tráfego para nova versão, risco muito baixo
- Immutable: novo ASG completo, troca quando saudável

#### CI/CD
- CodePipeline: orquestra o pipeline (source → build → test → deploy)
- CodeDeploy: deploy em EC2, ECS, Lambda com blue/green e canary
- CodeBuild: build + test serverless
- CloudFormation StackSets: IaC em múltiplas contas/regiões

#### CloudFormation
- StackSets: deploy multi-account/multi-region simultaneamente
- Nested Stacks: reutilização de componentes (módulos)
- Drift Detection: detecta mudanças manuais fora do template
- Change Sets: preview antes de aplicar
- DeletionPolicy: Retain, Snapshot, Delete
- Cross-stack references via Export/Import

#### Auto Scaling Groups
- Scaling Policies: Target Tracking ("CPU em 40%"), Step (alarm → add/remove), Scheduled, Predictive (ML forecast)
- Boas métricas: CPUUtilization, RequestCountPerTarget, Network In/Out, custom
- Health Checks: EC2 Status, ELB HTTP, Custom (SDK set-instance-health)
- Lifecycle Hooks: ações antes de in-service ou antes de terminate (cleanup, logs)
- Instance Refresh: atualiza launch template e recria EC2s (min healthy %)
- Scaling Processes (podem ser SUSPENSOS): Launch, Terminate, HealthCheck, ReplaceUnhealthy, AZRebalance

#### Spot Instances e Spot Fleets
- Até 90% desconto, 2 minutos aviso antes de interrupção
- Spot Fleet: conjunto de Spot + On-Demand, múltiplos launch pools
- Estratégias: lowestPrice (short), diversified (HA), capacityOptimized, priceCapacityOptimized (RECOMENDADO)
- Use para: batch, CI/CD, big data, workloads stateless

#### Systems Manager
- Session Manager: acesso remoto sem SSH/bastion (auditável)
- Patch Manager: patching automatizado com baselines
- Automation: runbooks operacionais
- Parameter Store: configs e secrets com hierarquia

### 2.2 Continuidade de Negócio

#### Active-Active vs Active-Passive
- Active-Active: ambas regiões servem, replicação bidirecional, failover automático, alto custo
- Active-Passive: apenas primária serve, replicação unidirecional, failover semi-automático, médio custo

#### Serviços Multi-Region Nativos
- DynamoDB Global Tables: multi-region active-active
- Aurora Global Database: 1 primária + até 5 secundárias, RPO < 1s
- S3 CRR: replicação assíncrona
- ElastiCache Global Datastore: Redis cross-region

### 2.3 Segurança para Novas Soluções

#### Proteção de Aplicações Web
- AWS WAF: SQLi, XSS, rate-limiting. Associado a CloudFront/ALB/API Gateway/AppSync
- Managed Rules: conjuntos pré-configurados (AWS, marketplace)
- Rate-based Rules: bloqueia IPs que excedem threshold
- Shield Standard: DDoS L3/L4 grátis automático
- Shield Advanced: $3000/mês, DRT team, L7, cost protection
- Network Firewall: IPS/IDS gerenciado na VPC, filtragem de domínio, stateful/stateless
- Firewall Manager: gerencia WAF/Shield/NFW/SGs org-wide

#### Criptografia em Trânsito
- ACM: certificados TLS gerenciados (grátis para serviços AWS)
- VPN: IPsec para Site-to-Site
- Client VPN: OpenVPN para acesso remoto

#### Criptografia em Repouso
- S3: SSE-S3 (padrão), SSE-KMS (auditável via CloudTrail), SSE-C (chave do cliente)
- EBS: KMS, account-level setting por região
- RDS: criptografa na criação APENAS. Para criptografar existente: snapshot → copy encrypted → restore
- DynamoDB: AWS owned key ou CMK

### 2.4 Confiabilidade

#### Mensageria e Desacoplamento
- SQS: queue pull-based, desacoplamento, buffering, DLQ para falhas
- SNS: pub/sub push-based, fan-out para múltiplos subscribers
- EventBridge: event bus, regras de roteamento, 18+ targets, archive/replay
- Kinesis Data Streams: real-time streaming, analytics
- Amazon MQ: broker gerenciado (ActiveMQ/RabbitMQ) para apps com MQTT/AMQP/STOMP
- Fan-out pattern: SNS → múltiplas SQS queues (processamento paralelo independente)

#### Step Functions
- Orquestração de workflows serverless (máquinas de estado)
- Standard: até 1 ano, exactly-once
- Express: até 5 minutos, at-least-once, alto throughput
- Integração nativa com Lambda, ECS, SNS, SQS, DynamoDB

### 2.5 Performance

#### Cache
- CloudFront: CDN edge, static + dynamic, Lambda@Edge
- API Gateway Cache: 0.5GB a 237GB
- ElastiCache Redis: Multi-AZ, replicação, persistência, rich data types, Global Datastore
- ElastiCache Memcached: simples, multi-thread, sem HA/replicação
- DAX: cache in-memory para DynamoDB (microsegundos)
- Redis = HA + persistência. Memcached = cache simples horizontal.

#### Databases - Escolha Correta
- Aurora: OLTP, MySQL/PostgreSQL compatible, 5x mais rápido, até 15 Read Replicas, Global Database
- DynamoDB: key-value, serverless, single-digit ms, Global Tables, DAX
- DocumentDB: compatível MongoDB
- Neptune: grafo (redes sociais, fraude)
- Timestream: time-series (IoT, métricas)
- QLDB: ledger imutável
- Keyspaces: compatível Cassandra
- OpenSearch: full-text search, log analytics
- "Milhões req/s com ms latência" → DynamoDB. "SQL com joins" → Aurora. "Relações" → Neptune.

### 2.6 Custos

#### S3 Storage Classes
- Standard: acesso frequente, ≥3 AZs
- Intelligent-Tiering: auto-move sem retrieval fee
- Standard-IA: infrequente, 30 dias min, 128KB min
- One Zone-IA: 1 AZ, dados recriáveis
- Glacier Instant: ms latência, 90 dias min
- Glacier Flexible: 1-12h (Expedited 1-5min), 90 dias min
- Glacier Deep Archive: 12-48h, 180 dias min
- Todas: 11 noves de durabilidade

#### Data Transfer Costs
- Ingress (entrada): GRÁTIS
- Egress (saída internet): cobra por GB
- Cross-AZ: $0.01/GB por direção
- Mesma AZ (IP privado): grátis
- Otimizações: VPC Gateway Endpoint (S3/DDB grátis), CloudFront (egress menor), Direct Connect (preço menor)

---

## DOMÍNIO 3: Melhoria Contínua (25%)

### 3.1 Excelência Operacional

#### Monitoramento
- CloudWatch Metrics: CPU, custom metrics, anomaly detection
- CloudWatch Logs: centralização, Insights para queries
- CloudWatch Alarms: threshold → SNS, Auto Scaling, EC2 actions
- CloudWatch Synthetics: canary scripts (monitora endpoints proativamente)
- X-Ray: tracing distribuído, análise visual de latência
- Cross-Account Observability: OAM (Observability Access Manager)

#### Automação e Remediação
- EventBridge + Lambda: reação a eventos AWS (SG aberto → Lambda fecha)
- AWS Config Rules + Remediation: detecta não-conformidade → SSM Automation corrige
- CloudWatch Alarm → Auto Scaling: escala baseado em métricas
- GuardDuty → EventBridge → Lambda: resposta automática a ameaças
- Systems Manager Automation: runbooks para tarefas operacionais
- Princípio na prova: automação > intervenção manual

### 3.2 Melhorar Segurança

#### Least Privilege
- IAM Access Analyzer Policy Generation: gera policy baseada em 90 dias de CloudTrail
- IAM Access Analyzer Policy Validation: valida contra grammar e best practices
- IAM Access Analyzer Findings: identifica recursos expostos externamente
- S3 Access Points: simplifica bucket policies complexas
- VPC Endpoint Policies: restringe quais recursos acessíveis pelo endpoint

#### Patching e Compliance
- SSM Patch Manager: patching automatizado com baselines
- Inspector: scan contínuo de vulnerabilidades
- Config Rules: detecta não-conformidade + auto-remediation
- GuardDuty → EventBridge → Lambda: resposta automática

### 3.3 Melhorar Performance

#### Rightsizing
- Compute Optimizer: recomenda instância ideal (métricas históricas de EC2, EBS, Lambda, ECS)
- Vertical Scaling: instância maior (limitado pelo tipo)
- Horizontal Scaling: Auto Scaling Groups
- Caching: CloudFront, ElastiCache, DAX
- Read Replicas: Aurora até 15 réplicas

#### EC2 Placement Groups
- Cluster: mesmo rack, menor latência (HPC, big data)
- Spread: hardware separado por instância (HA, max 7/AZ)
- Partition: racks separados por grupo (HDFS, Kafka, Cassandra)

### 3.4 Melhorar Confiabilidade

#### Eliminar Pontos Únicos de Falha
- Multi-AZ para stateful (RDS, ElastiCache, EFS)
- Auto Scaling com health checks adequados
- Loose coupling via SQS/SNS
- Dead Letter Queues para mensagens que falham
- Idempotência em processadores
- Circuit breaker para chamadas externas
- Graceful degradation

#### Self-Healing Patterns
- ASG + Health Checks: unhealthy → terminated → replaced
- ECS Service: task fail → new task launched
- Lambda: retry automático (2x async)
- RDS Multi-AZ: failover 30-60s automático
- Service Quotas: monitore com CloudWatch, request increase proativo

### 3.5 Redução de Custos

#### Identificando Desperdícios
- EBS volumes não anexados (você paga!)
- Elastic IPs não associados (cobra quando NÃO está em uso)
- Instâncias oversized (CPU < 10% por semanas)
- NAT Gateway com tráfego S3/DDB → VPC Endpoints
- S3 sem lifecycle policies
- Snapshots/AMIs antigos

#### Automação
- Instance Scheduler: liga/desliga EC2/RDS em horário comercial
- Budgets Actions: aplica SCP ou stop ao exceder budget
- Tagging obrigatório: SCP bloqueia criação sem tags de custo

---

## DOMÍNIO 4: Migração e Modernização (20%)

### 4.1 Os 7Rs da Migração
- Retire: desligar o que não precisa (economia 10-20%)
- Retain: manter onde está (compliance, dependências)
- Rehost (Lift & Shift): migrar como está (MGN), ~30% economia, sem mudança de código
- Relocate: VMware → VMware Cloud on AWS
- Replatform (Lift & Reshape): otimizações mínimas (MySQL on EC2 → RDS MySQL)
- Repurchase (Drop & Shop): trocar por SaaS (CRM → Salesforce)
- Refactor/Re-architect: redesenhar cloud-native (monólito → microsserviços)
- "Menor esforço" = Rehost. "Melhor uso da cloud" = Refactor.

### 4.2 Ferramentas de Migração

#### Assessment
- Migration Hub: dashboard centralizado de progresso
- Application Discovery Service: Agentless (VMware vCenter, config básica) ou Agent-based (processos, rede, dependências)
- Migration Evaluator: TCO, business case

#### Migração de Servidores
- Application Migration Service (MGN): rehost em larga escala, replicação contínua → test → cutover
- Substitui CloudEndure Migration e SMS
- VMware Cloud on AWS: para equipes expert em VMware

#### Migração de Banco de Dados
- DMS: Full Load + CDC (mínimo downtime), EC2 instance faz a replicação
- SCT: converte schema (só para heterogêneo, ex: Oracle → Aurora)
- Combinação típica: SCT (schema) + DMS (dados com CDC)
- DMS não replica OpenSearch (só como target)
- Snowball + DMS: SCT extrai → Snow ship → S3 → DMS migra (para volumes grandes)

#### Migração de Dados
- DataSync: NFS/SMB/HDFS → AWS (10Gbps/task, scheduling, preserva metadata)
- Snow Family: Snowball Edge Storage (210TB), Compute (28TB)
- Se > 1 semana pela rede = use Snow Family
- Transfer Family: SFTP/FTPS/FTP gerenciado → S3/EFS (manter workflows existentes)
- S3 Transfer Acceleration: upload via edge locations

#### Snow Family Performance Tips
- Múltiplos terminais em paralelo
- Agrupar arquivos pequenos (min 1MB)
- S3 Adapter: 250-400 MB/s vs File interface: 25-40 MB/s

### 4.3 Storage

#### EBS
- gp3: 3000 IOPS baseline, configurável até 16K
- io2 Block Express: até 256K IOPS, Multi-Attach (mesma AZ, cluster-aware FS)
- st1: throughput HDD, sc1: cold HDD
- Snapshots: incrementais, FSR para evitar latência primeiro acesso
- Data Lifecycle Manager: automação de snapshots por tags
- Encryption: account-level setting por região
- Instance Store: efêmero, milhões de IOPS, perdido no stop/terminate

#### EFS
- NFS multi-AZ, Linux only (POSIX), pay-per-use, escala para petabytes
- Performance Modes: General Purpose (latência) vs Max I/O (throughput paralelo)
- Throughput Modes: Bursting | Provisioned | Elastic (auto, até 3 GiB/s reads)
- Storage Classes: Standard → IA → Archive (50% mais barato), lifecycle policies
- One Zone: 90%+ economia, bom para dev
- Access Points: restringem acesso a diretórios com POSIX user/group
- Cross-Region Replication: RPO/RTO de minutos
- On-premises: via Direct Connect ou VPN (mount por IPv4)

#### FSx
- FSx for Windows: SMB + NTFS + AD + DFS Namespaces, Multi-AZ, backup daily S3
- FSx for Lustre: HPC/ML, integra com S3, Scratch (temp, burst 6x) vs Persistent (replicado)
- Lustre Lazy Loading: só carrega do S3 o que processa (reduz custo)
- FSx for NetApp ONTAP: NFS+SMB+iSCSI, shrink/grow auto, cloning
- FSx for OpenZFS: NFS, 1M IOPS, snapshots, cloning
- Migrar Single→Multi-AZ: DataSync ou backup/restore
- Reduzir volume: criar novo menor + DataSync

#### Storage Gateway
- S3 File Gateway: NFS/SMB → S3 (cache local, AD integration, Lifecycle para Glacier)
- Volume Gateway: iSCSI → EBS snapshots. Cached (recent data) vs Stored (full on-prem)
- Tape Gateway: VTL via iSCSI → S3 + Glacier (substitui fitas)
- FSx File Gateway: cache local de FSx Windows para branches
- File Gateway + S3 Object Lock: WORM data, RefreshCache API para restores

### 4.4 Modernização

#### Serverless Stack
- API Gateway (REST/HTTP/WebSocket) → Lambda → DynamoDB
- Cognito (auth), S3 (storage), Step Functions (orquestração), EventBridge (eventos)
- Lambda limits: 15min timeout, 10GB memória, 1000 concurrent (default), 10GB /tmp

#### Containers
- ECS: AWS-native, mais simples, sem custo control plane, Dynamic Port Mapping com ALB
- EKS: Kubernetes, portável, ecossistema K8s, $0.10/hr cluster
- Fargate: serverless containers (ECS ou EKS), paga vCPU/memória/segundo
- Fargate Spot: tasks reclamáveis (~70% desconto)
- awsvpc mode: ENI por task (SGs por task, default Fargate)
- ECS Service Auto Scaling: Application Auto Scaling (tasks) ≠ EC2 Auto Scaling (instâncias)
- ECR: registry privado, cross-region/cross-account replication, vulnerability scanning (Basic CVE ou Enhanced via Inspector)

---

## DICAS PARA A PROVA

### Estratégia de Resolução
1. Leia a ÚLTIMA frase primeiro (o que está sendo pedido)
2. Identifique os requisitos (multi-region, compliance, menor latência)
3. Elimine opções absurdas (serviço em contexto errado)
4. Entre 2 válidas: escolha a mais AWS-native (managed > self-managed)
5. "Menor esforço operacional" = serverless, managed services

### Gestão de Tempo
- 75 questões em 180 min = 2.4 min/questão
- Primeiro passe: responda o que sabe
- Segundo passe: revise marcadas
- NÃO gaste mais de 4 min em uma questão

### Padrões de Resposta
- "Menor esforço operacional" → managed services (Aurora > RDS > MySQL on EC2)
- "Menor custo" → pode ser custo imediato OU TCO (serverless vence TCO)
- "Menor tempo de migração" → Rehost > Replatform > Refactor
- "Mínimo downtime na migração" → DMS com CDC, Blue/Green
- "Compliance / regulatório" → KMS CMK, CloudTrail, Config, SCPs
- "Multi-region active-active" → DynamoDB Global Tables, Aurora Global
- "Loosely coupled" → SQS, SNS, EventBridge, Step Functions
- "Rotação de secrets" → Secrets Manager
- "Sem SSH/bastion" → Systems Manager Session Manager
- "Dados > 1 semana pela rede" → Snow Family
