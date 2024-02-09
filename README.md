# abstracta-smart-chatbot-sre


## 1- Create Infra with ARM:

### a- Docker Registry:
 Data required:
- registryName: The name of the Azure Container Registry.
- location: The location where the registry will be created. By default, it uses the location of the resource group.
- sku: The pricing tier (SKU) of the registry (Basic, Standard, or Premium).
- adminUserEnabled: Specifies whether the admin user for the registry is enabled or disabled.


      {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "registryName": {
            "type": "string",
            "metadata": {
              "description": "Name of the Azure Container Registry."
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          },
          "sku": {
            "type": "string",
            "defaultValue": "Basic",
            "allowedValues": [
              "Basic",
              "Standard",
              "Premium"
            ],
            "metadata": {
              "description": "The SKU of the container registry."
            }
          },
          "adminUserEnabled": {
            "type": "bool",
            "defaultValue": false,
            "metadata": {
              "description": "Enable admin user for the container registry."
            }
          }
        },
        "resources": [
          {
            "type": "Microsoft.ContainerRegistry/registries",
            "apiVersion": "2020-11-01-preview",
            "name": "[parameters('registryName')]",
            "location": "[parameters('location')]",
            "sku": {
              "name": "[parameters('sku')]"
            },
            "properties": {
              "adminUserEnabled": "[parameters('adminUserEnabled')]"
            }
          }
        ],
        "outputs": {
          "registryId": {
            "type": "string",
            "value": "[resourceId('Microsoft.ContainerRegistry/registries', parameters('registryName'))]"
          }
        }
      }
      
  
### b- DB
Data required:
- databaseName: The name of the Azure SQL Database.
- serverName: The name of the Azure SQL Server.
- administratorLogin: The administrator username for the Azure SQL Server.
- administratorLoginPassword: The password for the administrator username.
- collation: The collation for the Azure SQL Database.
- edition: The edition of the Azure SQL Database (Basic, Standard, or Premium).
- maxSizeBytes: The maximum size of the Azure SQL Database in bytes.

      {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "databaseName": {
            "type": "string",
            "metadata": {
              "description": "Name of the Azure SQL Database."
            }
          },
          "serverName": {
            "type": "string",
            "metadata": {
              "description": "Name of the Azure SQL Server."
            }
          },
          "administratorLogin": {
            "type": "string",
            "metadata": {
              "description": "Username for the Azure SQL Server."
            }
          },
          "administratorLoginPassword": {
            "type": "securestring",
            "metadata": {
              "description": "Password for the Azure SQL Server."
            }
          },
          "collation": {
            "type": "string",
            "defaultValue": "SQL_Latin1_General_CP1_CI_AS",
            "metadata": {
              "description": "Collation for the Azure SQL Database."
            }
          },
          "edition": {
            "type": "string",
            "defaultValue": "Basic",
            "allowedValues": [
              "Basic",
              "Standard",
              "Premium"
            ],
            "metadata": {
              "description": "Edition of the Azure SQL Database."
            }
          },
          "maxSizeBytes": {
            "type": "int",
            "defaultValue": 1073741824,
            "metadata": {
              "description": "Maximum size of the Azure SQL Database in bytes."
            }
          }
        },
        "resources": [
          {
            "type": "Microsoft.Sql/servers",
            "apiVersion": "2022-02-01-preview",
            "name": "[parameters('serverName')]",
            "location": "[resourceGroup().location]",
            "properties": {
              "administratorLogin": "[parameters('administratorLogin')]",
              "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
              "version": "12.0"
            },
            "resources": [
              {
                "type": "databases",
                "apiVersion": "2022-02-01-preview",
                "name": "[parameters('databaseName')]",
                "dependsOn": [
                  "[resourceId('Microsoft.Sql/servers', parameters('serverName'))]"
                ],
                "properties": {
                  "collation": "[parameters('collation')]",
                  "edition": "[parameters('edition')]",
                  "maxSizeBytes": "[parameters('maxSizeBytes')]",
                  "requestedServiceObjectiveName": "Basic"
                }
              }
            ]
          }
        ],
        "outputs": {
          "databaseResourceId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Sql/servers/databases', parameters('serverName'), parameters('databaseName'))]"
          }
        }
      }

### c- AKS
Data Required:
-aksClusterName: The name of the AKS cluster.
- location: The location where the cluster will be created. By default, it uses the location of the resource group.
- nodeCount: The number of nodes in the AKS cluster.
- vmSize: The size of the VMs in the AKS cluster node pool.
- kubernetesVersion: The version of Kubernetes.
- dnsPrefix: The DNS prefix for the AKS cluster. 


      {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "aksClusterName": {
            "type": "string",
            "metadata": {
              "description": "Name of the Azure Kubernetes Service (AKS) cluster."
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          },
          "nodeCount": {
            "type": "int",
            "defaultValue": 1,
            "metadata": {
              "description": "Number of nodes in the AKS cluster."
            }
          },
          "vmSize": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
              "description": "Size of the VMs in the AKS cluster node pool."
            }
          },
          "kubernetesVersion": {
            "type": "string",
            "defaultValue": "1.21.0",
            "metadata": {
              "description": "Version of Kubernetes."
            }
          },
          "dnsPrefix": {
            "type": "string",
            "metadata": {
              "description": "DNS prefix for the AKS cluster."
            }
          }
        },
        "resources": [
          {
            "type": "Microsoft.ContainerService/managedClusters",
            "apiVersion": "2021-07-01",
            "name": "[parameters('aksClusterName')]",
            "location": "[parameters('location')]",
            "properties": {
              "kubernetesVersion": "[parameters('kubernetesVersion')]",
              "enableRBAC": true,
              "networkProfile": {
                "networkPlugin": "azure",
                "serviceCidr": "10.0.0.0/16",
                "dnsServiceIP": "10.0.0.10",
                "dockerBridgeCidr": "172.17.0.1/16"
              },
              "agentPoolProfiles": [
                {
                  "name": "agentpool",
                  "count": "[parameters('nodeCount')]",
                  "vmSize": "[parameters('vmSize')]",
                  "osType": "Linux",
                  "mode": "System"
                }
              ]
            },
            "identity": {
              "type": "SystemAssigned"
            }
          }
        ],
        "outputs": {
          "kubeConfig": {
            "type": "string",
            "value": "[list(secrets('Microsoft.ContainerService/managedClusters', parameters('aksClusterName'), 'clusterAdminCredential').kubeconfigContent)[0]]"
          }
        }
      }



### d- Networking
Required data:
 - dnsZoneName: The name of the Azure DNS Zone.
 - aksClusterName: The name of the AKS cluster.
 - aksSubdomain: The subdomain for the AKS cluster DNS record.
 - sqlServerName: The name of the Azure SQL Server.
 - sqlServerSubdomain: The subdomain for the Azure SQL Server DNS record.
 - location: The location where the resources will be created. By default, it uses the location of the resource group.


          {
            "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
              "dnsZoneName": {
                "type": "string",
                "metadata": {
                  "description": "Name of the Azure DNS Zone."
                }
              },
              "aksClusterName": {
                "type": "string",
                "metadata": {
                  "description": "Name of the Azure Kubernetes Service (AKS) cluster."
                }
              },
              "aksSubdomain": {
                "type": "string",
                "metadata": {
                  "description": "Subdomain for the AKS cluster DNS record."
                }
              },
              "sqlServerName": {
                "type": "string",
                "metadata": {
                  "description": "Name of the Azure SQL Server."
                }
              },
              "sqlServerSubdomain": {
                "type": "string",
                "metadata": {
                  "description": "Subdomain for the Azure SQL Server DNS record."
                }
              },
              "location": {
                "type": "string",
                "defaultValue": "[resourceGroup().location]",
                "metadata": {
                  "description": "Location for all resources."
                }
              }
            },
            "resources": [
              {
                "type": "Microsoft.Network/dnszones",
                "apiVersion": "2018-05-01",
                "name": "[parameters('dnsZoneName')]",
                "location": "[parameters('location')]",
                "properties": {}
              },
              {
                "type": "Microsoft.Network/dnszones/A",
                "apiVersion": "2018-05-01",
                "name": "[concat(parameters('dnsZoneName'), '/', parameters('aksSubdomain'))]",
                "location": "[parameters('location')]",
                "dependsOn": [
                  "[resourceId('Microsoft.Network/dnszones', parameters('dnsZoneName'))]"
                ],
                "properties": {
                  "TTL": 300,
                  "ARecords": [
                    {
                      "ipv4Address": "[reference(resourceId('Microsoft.ContainerService/managedClusters', parameters('aksClusterName')), '2021-07-01', 'full').properties.fqdn]"
                    }
                  ]
                }
              },
              {
                "type": "Microsoft.Network/dnszones/A",
                "apiVersion": "2018-05-01",
                "name": "[concat(parameters('dnsZoneName'), '/', parameters('sqlServerSubdomain'))]",
                "location": "[parameters('location')]",
                "dependsOn": [
                  "[resourceId('Microsoft.Network/dnszones', parameters('dnsZoneName'))]"
                ],
                "properties": {
                  "TTL": 300,
                  "ARecords": [
                    {
                      "ipv4Address": "[reference(resourceId('Microsoft.Sql/servers', parameters('sqlServerName')), '2022-02-01', 'full').fullyQualifiedDomainName]"
                    }
                  ]
                }
              }
            ]
          }

## 2- Generate kube roles
a - developer                  (read only, pods, svc, ingresses)

b - devop                      (read/write, pods, svc, ingresses, ns, etc)

c - bot (for gitactions)       (read/write, pods, svc, ingresses, ns, etc)



## 3- Create CI/CD based on a simple gitflow:

  ![image](https://github.com/alangdcdevop/abstracta-smart-chatbot-sre/assets/50338530/d4d87e7c-ffea-49e1-811a-6883afdf3258)

### Rules:
  - If branch like "feature/..", run : - Sonar
  - If branch like "develop", run    :
    - Sonar
    - Automation Tests
    - Hadolint
    - Build image to registry
  - If branch like "feature/..", run : - Sonar
    - Deploy to AKS 

  ![image](https://github.com/alangdcdevop/abstracta-smart-chatbot-sre/assets/50338530/da1e7e25-801b-4d0b-8338-7da2f1b172ef)

4- Create logging infra in different ns:
![image](https://github.com/alangdcdevop/abstracta-smart-chatbot-sre/assets/50338530/44dd9017-5f92-43a1-a19e-66f63ee98a7d)


5- Configure deployment to check health check in aplication to reduce time lapse between a new pod is created.

6- Do stress test with jmeter or simiar tool to check current request tolerance.

Example of infrasctucture:
![image](https://github.com/alangdcdevop/abstracta-smart-chatbot-sre/assets/50338530/7f65807e-5123-4e31-aeb4-c6984fc864b0)

