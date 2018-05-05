# 更新 Channel 配置

## 什么是 Channel 配置?

Channel配置包含所有与Channel管理相关的信息, 它包含Organization 和 Channel的隶属关系, Channel 访问策略, 区块大小等. 

这些配置信息存储在共享账本的区块中，称为配置块(CONFIG_BLOCK). 配置块包含单个配置, 其中, 第一个区块(配置块)被称为“天使块”，它包含引导Channel所需的初始配置信息。Channel的每次更改，都是通过新配置块的方式完成，其中最新的配置块代表当前Channel的配置信息。Orderers和Peers将当前Channel配置信息保持在内存中，以便于促进Channel操作，例如剪切新块和验证块事务等。

因为配置信息存储在区块中，所以更新Channel配置, 是通过提交“配置事务”完成的. 一般过程是,先获取Channel配置,然后将其转换成人类可读的格式并修改相应配置,最后提交一个修改事务.

想更深入地了解配置和JSON的转换过程，请参考[添加Organiaztion](https://linkto)。在这个文档中，我们将重点介绍不同的方式来编辑配置和获得签名的过程。

## 编辑配置

Channel是高度可配置的，但不是无限的。不同的配置元素具有不同的修改策略（指对更新配置进行签名所需的标识组).

要了解哪些域可以修改，我们需要先了解配置的JSON格式。[添加Organiaztion](https://linkto)中介绍了如何生成配置的JSON格式，所以你已经阅读过该章节, 那么你可以直接引用它。对于哪些还没有了解这些内容的人，我们将在这里提供一个示。

````
{
"channel_group": {
  "groups": {
    "Application": {
      "groups": {
        "Org1MSP": {
          "mod_policy": "Admins",
          "policies": {
            "Admins": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org1MSP",
                        "role": "ADMIN"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Readers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org1MSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Writers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org1MSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            }
          },
          "values": {
            "AnchorPeers": {
              "mod_policy": "Admins",
              "value": {
                "anchor_peers": [
                  {
                    "host": "peer0.org1.example.com",
                    "port": 7051
                  }
                ]
              },
              "version": "0"
            },
            "MSP": {
              "mod_policy": "Admins",
              "value": {
                "config": {
                  "admins": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNHRENDQWIrZ0F3SUJBZ0lRSWlyVmg3NVcwWmh0UjEzdmltdmliakFLQmdncWhrak9QUVFEQWpCek1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVpNQmNHQTFVRUNoTVFiM0puTVM1bGVHRnRjR3hsTG1OdmJURWNNQm9HQTFVRUF4TVRZMkV1CmIzSm5NUzVsZUdGdGNHeGxMbU52YlRBZUZ3MHhOekV4TWpreE9USTBNRFphRncweU56RXhNamN4T1RJME1EWmEKTUZzeEN6QUpCZ05WQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVApZVzRnUm5KaGJtTnBjMk52TVI4d0hRWURWUVFEREJaQlpHMXBia0J2Y21jeExtVjRZVzF3YkdVdVkyOXRNRmt3CkV3WUhLb1pJemowQ0FRWUlLb1pJemowREFRY0RRZ0FFNkdVeDlpczZ0aG1ZRE9tMmVHSlA5eW1yaXJYWE1Cd0oKQmVWb1Vpak5haUdsWE03N2NsSE5aZjArMGFjK2djRU5lMzQweGExZVFnb2Q0YjVFcmQrNmtxTk5NRXN3RGdZRApWUjBQQVFIL0JBUURBZ2VBTUF3R0ExVWRFd0VCL3dRQ01BQXdLd1lEVlIwakJDUXdJb0FnWWdoR2xCMjBGWmZCCllQemdYT280czdkU1k1V3NKSkRZbGszTDJvOXZzQ013Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnYmlEWDVTMlIKRTBNWGRobDZFbmpVNm1lTEJ0eXNMR2ZpZXZWTlNmWW1UQVVDSUdVbnROangrVXZEYkZPRHZFcFRVTm5MUHp0Qwp5ZlBnOEhMdWpMaXVpaWFaCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
                  ],
                  "crypto_config": {
                    "identity_identifier_hash_function": "SHA256",
                    "signature_hash_family": "SHA2"
                  },
                  "name": "Org1MSP",
                  "root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNRekNDQWVxZ0F3SUJBZ0lSQU03ZVdTaVM4V3VVM2haMU9tR255eXd3Q2dZSUtvWkl6ajBFQXdJd2N6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHVEFYQmdOVkJBb1RFRzl5WnpFdVpYaGhiWEJzWlM1amIyMHhIREFhQmdOVkJBTVRFMk5oCkxtOXlaekV1WlhoaGJYQnNaUzVqYjIwd0hoY05NVGN4TVRJNU1Ua3lOREEyV2hjTk1qY3hNVEkzTVRreU5EQTIKV2pCek1Rc3dDUVlEVlFRR0V3SlZVekVUTUJFR0ExVUVDQk1LUTJGc2FXWnZjbTVwWVRFV01CUUdBMVVFQnhNTgpVMkZ1SUVaeVlXNWphWE5qYnpFWk1CY0dBMVVFQ2hNUWIzSm5NUzVsZUdGdGNHeGxMbU52YlRFY01Cb0dBMVVFCkF4TVRZMkV1YjNKbk1TNWxlR0Z0Y0d4bExtTnZiVEJaTUJNR0J5cUdTTTQ5QWdFR0NDcUdTTTQ5QXdFSEEwSUEKQkJiTTVZS3B6UmlEbDdLWWFpSDVsVnBIeEl1TDEyaUcyWGhkMHRpbEg3MEljMGFpRUh1dG9rTkZsUXAzTWI0Zgpvb0M2bFVXWnRnRDJwMzZFNThMYkdqK2pYekJkTUE0R0ExVWREd0VCL3dRRUF3SUJwakFQQmdOVkhTVUVDREFHCkJnUlZIU1VBTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME9CQ0lFSUdJSVJwUWR0QldYd1dEODRGenEKT0xPM1VtT1ZyQ1NRMkpaTnk5cVBiN0FqTUFvR0NDcUdTTTQ5QkFNQ0EwY0FNRVFDSUdlS2VZL1BsdGlWQTRPSgpRTWdwcDRvaGRMcGxKUFpzNERYS0NuOE9BZG9YQWlCK2g5TFdsR3ZsSDdtNkVpMXVRcDFld2ZESmxsZi9MZXczClgxaDNRY0VMZ3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
                  ],
                  "tls_root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTVENDQWZDZ0F3SUJBZ0lSQUtsNEFQWmV6dWt0Nk8wYjRyYjY5Y0F3Q2dZSUtvWkl6ajBFQXdJd2RqRUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHVEFYQmdOVkJBb1RFRzl5WnpFdVpYaGhiWEJzWlM1amIyMHhIekFkQmdOVkJBTVRGblJzCmMyTmhMbTl5WnpFdVpYaGhiWEJzWlM1amIyMHdIaGNOTVRjeE1USTVNVGt5TkRBMldoY05NamN4TVRJM01Ua3kKTkRBMldqQjJNUXN3Q1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRQpCeE1OVTJGdUlFWnlZVzVqYVhOamJ6RVpNQmNHQTFVRUNoTVFiM0puTVM1bGVHRnRjR3hsTG1OdmJURWZNQjBHCkExVUVBeE1XZEd4elkyRXViM0puTVM1bGVHRnRjR3hsTG1OdmJUQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDkKQXdFSEEwSUFCSnNpQXVjYlcrM0lqQ2VaaXZPakRiUmFyVlRjTW9TRS9mSnQyU0thR1d5bWQ0am5xM25MWC9vVApCVmpZb21wUG1QbGZ4R0VSWHl0UTNvOVZBL2hwNHBlalh6QmRNQTRHQTFVZER3RUIvd1FFQXdJQnBqQVBCZ05WCkhTVUVDREFHQmdSVkhTVUFNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJSnlqZnFoa0FvY3oKdkRpNnNGSGFZL1Bvd2tPWkxPMHZ0VGdFRnVDbUpFalZNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJRjVOVVdCVgpmSjgrM0lxU3J1NlFFbjlIa0lsQ0xDMnlvWTlaNHBWMnpBeFNBaUE5NWQzeDhBRXZIcUFNZnIxcXBOWHZ1TW5BCmQzUXBFa1gyWkh3ODZlQlVQZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
                  ]
                },
                "type": 0
              },
              "version": "0"
            }
          },
          "version": "1"
        },
        "Org2MSP": {
          "mod_policy": "Admins",
          "policies": {
            "Admins": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org2MSP",
                        "role": "ADMIN"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Readers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org2MSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Writers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org2MSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            }
          },
          "values": {
            "AnchorPeers": {
              "mod_policy": "Admins",
              "value": {
                "anchor_peers": [
                  {
                    "host": "peer0.org2.example.com",
                    "port": 7051
                  }
                ]
              },
              "version": "0"
            },
            "MSP": {
              "mod_policy": "Admins",
              "value": {
                "config": {
                  "admins": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNHVENDQWNDZ0F3SUJBZ0lSQU5Pb1lIbk9seU94dTJxZFBteStyV293Q2dZSUtvWkl6ajBFQXdJd2N6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHVEFYQmdOVkJBb1RFRzl5WnpJdVpYaGhiWEJzWlM1amIyMHhIREFhQmdOVkJBTVRFMk5oCkxtOXlaekl1WlhoaGJYQnNaUzVqYjIwd0hoY05NVGN4TVRJNU1Ua3lOREEyV2hjTk1qY3hNVEkzTVRreU5EQTIKV2pCYk1Rc3dDUVlEVlFRR0V3SlZVekVUTUJFR0ExVUVDQk1LUTJGc2FXWnZjbTVwWVRFV01CUUdBMVVFQnhNTgpVMkZ1SUVaeVlXNWphWE5qYnpFZk1CMEdBMVVFQXd3V1FXUnRhVzVBYjNKbk1pNWxlR0Z0Y0d4bExtTnZiVEJaCk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQkh1M0ZWMGlqdFFzckpsbnBCblgyRy9ickFjTHFJSzgKVDFiSWFyZlpvSkhtQm5IVW11RTBhc1dyKzM4VUs0N3hyczNZMGMycGhFVjIvRnhHbHhXMUZubWpUVEJMTUE0RwpBMVVkRHdFQi93UUVBd0lIZ0RBTUJnTlZIUk1CQWY4RUFqQUFNQ3NHQTFVZEl3UWtNQ0tBSU1pSzdteFpnQVVmCmdrN0RPTklXd2F4YktHVGdLSnVSNjZqVmordHZEV3RUTUFvR0NDcUdTTTQ5QkFNQ0EwY0FNRVFDSUQxaEtRdk8KVWxyWmVZMmZZY1N2YWExQmJPM3BVb3NxL2tZVElyaVdVM1J3QWlBR29mWmVPUFByWXVlTlk0Z2JCV2tjc3lpZgpNMkJmeXQwWG9NUThyT2VidUE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
                  ],
                  "crypto_config": {
                    "identity_identifier_hash_function": "SHA256",
                    "signature_hash_family": "SHA2"
                  },
                  "name": "Org2MSP",
                  "root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNSRENDQWVxZ0F3SUJBZ0lSQU1pVXk5SGRSbXB5MDdsSjhRMlZNWXN3Q2dZSUtvWkl6ajBFQXdJd2N6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHVEFYQmdOVkJBb1RFRzl5WnpJdVpYaGhiWEJzWlM1amIyMHhIREFhQmdOVkJBTVRFMk5oCkxtOXlaekl1WlhoaGJYQnNaUzVqYjIwd0hoY05NVGN4TVRJNU1Ua3lOREEyV2hjTk1qY3hNVEkzTVRreU5EQTIKV2pCek1Rc3dDUVlEVlFRR0V3SlZVekVUTUJFR0ExVUVDQk1LUTJGc2FXWnZjbTVwWVRFV01CUUdBMVVFQnhNTgpVMkZ1SUVaeVlXNWphWE5qYnpFWk1CY0dBMVVFQ2hNUWIzSm5NaTVsZUdGdGNHeGxMbU52YlRFY01Cb0dBMVVFCkF4TVRZMkV1YjNKbk1pNWxlR0Z0Y0d4bExtTnZiVEJaTUJNR0J5cUdTTTQ5QWdFR0NDcUdTTTQ5QXdFSEEwSUEKQk50YW1PY1hyaGwrQ2hzYXNSeklNWjV3OHpPWVhGcXhQbGV0a3d5UHJrbHpKWE01Qjl4QkRRVWlWNldJS2tGSwo0Vmd5RlNVWGZqaGdtd25kMUNBVkJXaWpYekJkTUE0R0ExVWREd0VCL3dRRUF3SUJwakFQQmdOVkhTVUVDREFHCkJnUlZIU1VBTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME9CQ0lFSU1pSzdteFpnQVVmZ2s3RE9OSVcKd2F4YktHVGdLSnVSNjZqVmordHZEV3RUTUFvR0NDcUdTTTQ5QkFNQ0EwZ0FNRVVDSVFEQ3FFRmFqeU5IQmVaRworOUdWVkNFNWI1YTF5ZlhvS3lkemdLMVgyOTl4ZmdJZ05BSUUvM3JINHFsUE9HbjdSS3Yram9WaUNHS2t6L0F1Cm9FNzI4RWR6WmdRPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
                  ],
                  "tls_root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTakNDQWZDZ0F3SUJBZ0lSQU9JNmRWUWMraHBZdkdMSlFQM1YwQU13Q2dZSUtvWkl6ajBFQXdJd2RqRUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHVEFYQmdOVkJBb1RFRzl5WnpJdVpYaGhiWEJzWlM1amIyMHhIekFkQmdOVkJBTVRGblJzCmMyTmhMbTl5WnpJdVpYaGhiWEJzWlM1amIyMHdIaGNOTVRjeE1USTVNVGt5TkRBMldoY05NamN4TVRJM01Ua3kKTkRBMldqQjJNUXN3Q1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRQpCeE1OVTJGdUlFWnlZVzVqYVhOamJ6RVpNQmNHQTFVRUNoTVFiM0puTWk1bGVHRnRjR3hsTG1OdmJURWZNQjBHCkExVUVBeE1XZEd4elkyRXViM0puTWk1bGVHRnRjR3hsTG1OdmJUQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDkKQXdFSEEwSUFCTWZ1QTMwQVVBT1ZKRG1qVlBZd1lNbTlweW92MFN6OHY4SUQ5N0twSHhXOHVOOUdSOU84aVdFMgo5bllWWVpiZFB2V1h1RCszblpweUFNcGZja3YvYUV5alh6QmRNQTRHQTFVZER3RUIvd1FFQXdJQnBqQVBCZ05WCkhTVUVDREFHQmdSVkhTVUFNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJRnk5VHBHcStQL08KUGRXbkZXdWRPTnFqVDRxOEVKcDJmbERnVCtFV2RnRnFNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUUNZYlhSeApXWDZoUitPU0xBNSs4bFRwcXRMWnNhOHVuS3J3ek1UYXlQUXNVd0lnVSs5YXdaaE0xRzg3bGE0V0h4cmt5eVZ2CkU4S1ZsR09IVHVPWm9TMU5PT0U9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
                  ]
                },
                "type": 0
              },
              "version": "0"
            }
          },
          "version": "1"
        },
        "Org3MSP": {
          "groups": {},
          "mod_policy": "Admins",
          "policies": {
            "Admins": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org3MSP",
                        "role": "ADMIN"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Readers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org3MSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Writers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "Org3MSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            }
          },
          "values": {
            "MSP": {
              "mod_policy": "Admins",
              "value": {
                "config": {
                  "admins": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNHRENDQWIrZ0F3SUJBZ0lRQUlSNWN4U0hpVm1kSm9uY3FJVUxXekFLQmdncWhrak9QUVFEQWpCek1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVpNQmNHQTFVRUNoTVFiM0puTXk1bGVHRnRjR3hsTG1OdmJURWNNQm9HQTFVRUF4TVRZMkV1CmIzSm5NeTVsZUdGdGNHeGxMbU52YlRBZUZ3MHhOekV4TWpreE9UTTRNekJhRncweU56RXhNamN4T1RNNE16QmEKTUZzeEN6QUpCZ05WQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVApZVzRnUm5KaGJtTnBjMk52TVI4d0hRWURWUVFEREJaQlpHMXBia0J2Y21jekxtVjRZVzF3YkdVdVkyOXRNRmt3CkV3WUhLb1pJemowQ0FRWUlLb1pJemowREFRY0RRZ0FFSFlkVFY2ZC80cmR4WFd2cm1qZ0hIQlhXc2lxUWxrcnQKZ0p1NzMxcG0yZDRrWU82aEd2b2tFRFBwbkZFdFBwdkw3K1F1UjhYdkFQM0tqTkt0NHdMRG5hTk5NRXN3RGdZRApWUjBQQVFIL0JBUURBZ2VBTUF3R0ExVWRFd0VCL3dRQ01BQXdLd1lEVlIwakJDUXdJb0FnSWNxUFVhM1VQNmN0Ck9LZmYvKzVpMWJZVUZFeVFlMVAyU0hBRldWSWUxYzB3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnUm5LRnhsTlYKSmppVGpkZmVoczRwNy9qMkt3bFVuUWVuNFkyUnV6QjFrbm9DSUd3dEZ1TEdpRFY2THZSL2pHVXR3UkNyeGw5ZApVNENCeDhGbjBMdXNMTkJYCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
                  ],
                  "crypto_config": {
                    "identity_identifier_hash_function": "SHA256",
                    "signature_hash_family": "SHA2"
                  },
                  "name": "Org3MSP",
                  "root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNRakNDQWVtZ0F3SUJBZ0lRUkN1U2Y0RVJNaDdHQW1ydTFIQ2FZREFLQmdncWhrak9QUVFEQWpCek1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVpNQmNHQTFVRUNoTVFiM0puTXk1bGVHRnRjR3hsTG1OdmJURWNNQm9HQTFVRUF4TVRZMkV1CmIzSm5NeTVsZUdGdGNHeGxMbU52YlRBZUZ3MHhOekV4TWpreE9UTTRNekJhRncweU56RXhNamN4T1RNNE16QmEKTUhNeEN6QUpCZ05WQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVApZVzRnUm5KaGJtTnBjMk52TVJrd0Z3WURWUVFLRXhCdmNtY3pMbVY0WVcxd2JHVXVZMjl0TVJ3d0dnWURWUVFECkV4TmpZUzV2Y21jekxtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkl6ajBEQVFjRFFnQUUKZXFxOFFQMnllM08vM1J3UzI0SWdtRVdST3RnK3Zyc2pRY1BvTU42NEZiUGJKbmExMklNaVdDUTF6ZEZiTU9hSAorMUlrb21yY0RDL1ZpejkvY0M0NW9xTmZNRjB3RGdZRFZSMFBBUUgvQkFRREFnR21NQThHQTFVZEpRUUlNQVlHCkJGVWRKUUF3RHdZRFZSMFRBUUgvQkFVd0F3RUIvekFwQmdOVkhRNEVJZ1FnSWNxUFVhM1VQNmN0T0tmZi8rNWkKMWJZVUZFeVFlMVAyU0hBRldWSWUxYzB3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnTEgxL2xSZElWTVA4Z2FWeQpKRW01QWQ0SjhwZ256N1BVV2JIMzZvdVg4K1lDSUNPK20vUG9DbDRIbTlFbXhFN3ZnUHlOY2trVWd0SlRiTFhqCk5SWjBxNTdWCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
                  ],
                  "tls_root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTVENDQWZDZ0F3SUJBZ0lSQU9xc2JQQzFOVHJzclEvUUNpalh6K0F3Q2dZSUtvWkl6ajBFQXdJd2RqRUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHVEFYQmdOVkJBb1RFRzl5WnpNdVpYaGhiWEJzWlM1amIyMHhIekFkQmdOVkJBTVRGblJzCmMyTmhMbTl5WnpNdVpYaGhiWEJzWlM1amIyMHdIaGNOTVRjeE1USTVNVGt6T0RNd1doY05NamN4TVRJM01Ua3oKT0RNd1dqQjJNUXN3Q1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRQpCeE1OVTJGdUlFWnlZVzVqYVhOamJ6RVpNQmNHQTFVRUNoTVFiM0puTXk1bGVHRnRjR3hsTG1OdmJURWZNQjBHCkExVUVBeE1XZEd4elkyRXViM0puTXk1bGVHRnRjR3hsTG1OdmJUQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDkKQXdFSEEwSUFCSVJTTHdDejdyWENiY0VLMmhxSnhBVm9DaDhkejNqcnA5RHMyYW9TQjBVNTZkSUZhVmZoR2FsKwovdGp6YXlndXpFalFhNlJ1MmhQVnRGM2NvQnJ2Ulpxalh6QmRNQTRHQTFVZER3RUIvd1FFQXdJQnBqQVBCZ05WCkhTVUVDREFHQmdSVkhTVUFNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJQ2FkVERGa0JPTGkKblcrN2xCbDExL3pPbXk4a1BlYXc0MVNZWEF6cVhnZEVNQW9HQ0NxR1NNNDlCQU1DQTBjQU1FUUNJQlgyMWR3UwpGaG5NdDhHWXUweEgrUGd5aXQreFdQUjBuTE1Jc1p2dVlRaktBaUFLUlE5N2VrLzRDTzZPWUtSakR0VFM4UFRmCm9nTmJ6dTBxcThjbVhseW5jZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
                  ]
                },
                "type": 0
              },
              "version": "0"
            }
          },
          "version": "0"
        }
      },
      "mod_policy": "Admins",
      "policies": {
        "Admins": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "MAJORITY",
              "sub_policy": "Admins"
            }
          },
          "version": "0"
        },
        "Readers": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "ANY",
              "sub_policy": "Readers"
            }
          },
          "version": "0"
        },
        "Writers": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "ANY",
              "sub_policy": "Writers"
            }
          },
          "version": "0"
        }
      },
      "version": "1"
    },
    "Orderer": {
      "groups": {
        "OrdererOrg": {
          "mod_policy": "Admins",
          "policies": {
            "Admins": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "OrdererMSP",
                        "role": "ADMIN"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Readers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "OrdererMSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            },
            "Writers": {
              "mod_policy": "Admins",
              "policy": {
                "type": 1,
                "value": {
                  "identities": [
                    {
                      "principal": {
                        "msp_identifier": "OrdererMSP",
                        "role": "MEMBER"
                      },
                      "principal_classification": "ROLE"
                    }
                  ],
                  "rule": {
                    "n_out_of": {
                      "n": 1,
                      "rules": [
                        {
                          "signed_by": 0
                        }
                      ]
                    }
                  },
                  "version": 0
                }
              },
              "version": "0"
            }
          },
          "values": {
            "MSP": {
              "mod_policy": "Admins",
              "value": {
                "config": {
                  "admins": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNDakNDQWJDZ0F3SUJBZ0lRSFNTTnIyMWRLTTB6THZ0dEdoQnpMVEFLQmdncWhrak9QUVFEQWpCcE1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4RnpBVkJnTlZCQU1URG1OaExtVjRZVzF3CmJHVXVZMjl0TUI0WERURTNNVEV5T1RFNU1qUXdObG9YRFRJM01URXlOekU1TWpRd05sb3dWakVMTUFrR0ExVUUKQmhNQ1ZWTXhFekFSQmdOVkJBZ1RDa05oYkdsbWIzSnVhV0V4RmpBVUJnTlZCQWNURFZOaGJpQkdjbUZ1WTJsegpZMjh4R2pBWUJnTlZCQU1NRVVGa2JXbHVRR1Y0WVcxd2JHVXVZMjl0TUZrd0V3WUhLb1pJemowQ0FRWUlLb1pJCnpqMERBUWNEUWdBRTZCTVcvY0RGUkUvakFSenV5N1BjeFQ5a3pnZitudXdwKzhzK2xia0hZd0ZpaForMWRhR3gKKzhpS1hDY0YrZ0tpcVBEQXBpZ2REOXNSeTBoTEMwQnRacU5OTUVzd0RnWURWUjBQQVFIL0JBUURBZ2VBTUF3RwpBMVVkRXdFQi93UUNNQUF3S3dZRFZSMGpCQ1F3SW9BZ3o3bDQ2ZXRrODU0NFJEanZENVB6YjV3TzI5N0lIMnNUCngwTjAzOHZibkpzd0NnWUlLb1pJemowRUF3SURTQUF3UlFJaEFNRTJPWXljSnVyYzhVY2hkeTA5RU50RTNFUDIKcVoxSnFTOWVCK0gxSG5FSkFpQUtXa2h5TmI0akRPS2MramJIVmgwV0YrZ3J4UlJYT1hGaEl4ei85elI3UUE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
                  ],
                  "crypto_config": {
                    "identity_identifier_hash_function": "SHA256",
                    "signature_hash_family": "SHA2"
                  },
                  "name": "OrdererMSP",
                  "root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNMakNDQWRXZ0F3SUJBZ0lRY2cxUVZkVmU2Skd6YVU1cmxjcW4vakFLQmdncWhrak9QUVFEQWpCcE1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4RnpBVkJnTlZCQU1URG1OaExtVjRZVzF3CmJHVXVZMjl0TUI0WERURTNNVEV5T1RFNU1qUXdObG9YRFRJM01URXlOekU1TWpRd05sb3dhVEVMTUFrR0ExVUUKQmhNQ1ZWTXhFekFSQmdOVkJBZ1RDa05oYkdsbWIzSnVhV0V4RmpBVUJnTlZCQWNURFZOaGJpQkdjbUZ1WTJsegpZMjh4RkRBU0JnTlZCQW9UQzJWNFlXMXdiR1V1WTI5dE1SY3dGUVlEVlFRREV3NWpZUzVsZUdGdGNHeGxMbU52CmJUQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJQTVI2MGdCcVJham9hS0U1TExRYjRIb28wN3QKYTRuM21Ncy9NRGloQVQ5YUN4UGZBcDM5SS8wMmwvZ2xiMTdCcEtxZGpGd0JKZHNuMVN6ZnQ3NlZkTitqWHpCZApNQTRHQTFVZER3RUIvd1FFQXdJQnBqQVBCZ05WSFNVRUNEQUdCZ1JWSFNVQU1BOEdBMVVkRXdFQi93UUZNQU1CCkFmOHdLUVlEVlIwT0JDSUVJTSs1ZU9uclpQT2VPRVE0N3crVDgyK2NEdHZleUI5ckU4ZERkTi9MMjV5Yk1Bb0cKQ0NxR1NNNDlCQU1DQTBjQU1FUUNJQVB6SGNOUmQ2a3QxSEdpWEFDclFTM0grL3R5NmcvVFpJa1pTeXIybmdLNQpBaUJnb1BVTTEwTHNsMVFtb2dlbFBjblZGZjJoODBXR2I3NGRIS2tzVFJKUkx3PT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
                  ],
                  "tls_root_certs": [
                    "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNORENDQWR1Z0F3SUJBZ0lRYWJ5SUl6cldtUFNzSjJacisvRVpXVEFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEUzTVRFeU9URTVNalF3TmxvWERUSTNNVEV5TnpFNU1qUXdObG93YkRFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEZEQVNCZ05WQkFvVEMyVjRZVzF3YkdVdVkyOXRNUm93R0FZRFZRUURFeEYwYkhOallTNWxlR0Z0CmNHeGxMbU52YlRCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQkVZVE9mdG1rTHdiSlRNeG1aVzMKZVdqRUQ2eW1UeEhYeWFQdTM2Y1NQWDlldDZyU3Y5UFpCTGxyK3hZN1dtYlhyOHM5K3E1RDMwWHl6OEh1OWthMQpSc1dqWHpCZE1BNEdBMVVkRHdFQi93UUVBd0lCcGpBUEJnTlZIU1VFQ0RBR0JnUlZIU1VBTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlJcjduNTVjTWlUdENEYmM5UGU0RFpnZ0ZYdHV2RktTdnBNYUhzbzAKSnpFd01Bb0dDQ3FHU000OUJBTUNBMGNBTUVRQ0lGM1gvMGtQRkFVQzV2N25JVVh6SmI5Z3JscWxET05UeVg2QQpvcmtFVTdWb0FpQkpMbS9IUFZ0aVRHY2NldUZPZTE4SnNwd0JTZ1hxNnY1K1BobEdsbU9pWHc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
                  ]
                },
                "type": 0
              },
              "version": "0"
            }
          },
          "version": "0"
        }
      },
      "mod_policy": "Admins",
      "policies": {
        "Admins": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "MAJORITY",
              "sub_policy": "Admins"
            }
          },
          "version": "0"
        },
        "BlockValidation": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "ANY",
              "sub_policy": "Writers"
            }
          },
          "version": "0"
        },
        "Readers": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "ANY",
              "sub_policy": "Readers"
            }
          },
          "version": "0"
        },
        "Writers": {
          "mod_policy": "Admins",
          "policy": {
            "type": 3,
            "value": {
              "rule": "ANY",
              "sub_policy": "Writers"
            }
          },
          "version": "0"
        }
      },
      "values": {
        "BatchSize": {
          "mod_policy": "Admins",
          "value": {
            "absolute_max_bytes": 103809024,
            "max_message_count": 10,
            "preferred_max_bytes": 524288
          },
          "version": "0"
        },
        "BatchTimeout": {
          "mod_policy": "Admins",
          "value": {
            "timeout": "2s"
          },
          "version": "0"
        },
        "ChannelRestrictions": {
          "mod_policy": "Admins",
          "version": "0"
        },
        "ConsensusType": {
          "mod_policy": "Admins",
          "value": {
            "type": "solo"
          },
          "version": "0"
        }
      },
      "version": "0"
    }
  },
  "mod_policy": "",
  "policies": {
    "Admins": {
      "mod_policy": "Admins",
      "policy": {
        "type": 3,
        "value": {
          "rule": "MAJORITY",
          "sub_policy": "Admins"
        }
      },
      "version": "0"
    },
    "Readers": {
      "mod_policy": "Admins",
      "policy": {
        "type": 3,
        "value": {
          "rule": "ANY",
          "sub_policy": "Readers"
        }
      },
      "version": "0"
    },
    "Writers": {
      "mod_policy": "Admins",
      "policy": {
        "type": 3,
        "value": {
          "rule": "ANY",
          "sub_policy": "Writers"
        }
      },
      "version": "0"
    }
  },
  "values": {
    "BlockDataHashingStructure": {
      "mod_policy": "Admins",
      "value": {
        "width": 4294967295
      },
      "version": "0"
    },
    "Consortium": {
      "mod_policy": "Admins",
      "value": {
        "name": "SampleConsortium"
      },
      "version": "0"
    },
    "HashingAlgorithm": {
      "mod_policy": "Admins",
      "value": {
        "name": "SHA256"
      },
      "version": "0"
    },
    "OrdererAddresses": {
      "mod_policy": "/Channel/Orderer/Admins",
      "value": {
        "addresses": [
          "orderer.example.com:7050"
        ]
      },
      "version": "0"
    }
  },
  "version": "0"
},
"sequence": "3",
"type": 0
}
````
配置可能看起来很吓人，但是一旦你研究它，你会发现它有一个逻辑结构。

除了策略的定义(定义谁可以在Channel级做什么事情，以及谁有权改变配置), Channel还可以使用更新配置来修改其他类型的特性, 例如[添加Organiaztion](https://linkto)介绍了如何向Channel添加新的Organization。其它可以用更新配置来修改的东西包括:

* **Batch Size** 这些参数指定了Block中Transaction的数量和大小。其中, Block的大小由````absolute_max_bytes````指定, Block中Transaction的最大数量由````max_message_count````指定, 如果指定了````preferred_max_bytes````, 那么在可能的情况下,区块会被切分.
    ````
    {
    "absolute_max_bytes": 102760448,
    "max_message_count": 10,
    "preferred_max_bytes": 524288
    }
    ````
* **Batch Timeout** 该参数制定了第一个Transaction到达之后到切分Block之前需要等待的时间。减小该值改进等待时间，但如果减小过多,会降低吞吐量.
    ````
    { "timeout": "2s" }
    ````
* **Channel Restrictions** Orderer可以分配的Channel总数,可以通过````max_count````指定.这主要适用于弱共识````ChannelCreation````策略的预生产环境中。
    ````
    { "max_count": 1000 }
    ````
* **Channel Creation Policy** 该参数用于指定Channel创建策略。Channel创建请求中附加的签名信息,会基于该策略的实例进行验证,以确保创建Channel的操作是已授权的. 注意:该参数只能使用在````orderer system channel````
    ````
    {
        "type": 3,
        "value": {
            "rule": "ANY",
            "sub_policy": "Admins"
        }
    }
    ````
* **Kafka brokers** 当````ConsensusType````被设置为````kafka````时，````brokers````列表枚举了````brokers````的一些子集，以便````orderer````在启动时连接到它们。
    ````
    {
        "brokers": [
            "kafka0:9092",
            "kafka1:9092",
            "kafka2:9092",
            "kafka3:9092"
        ]
    }
    ````


* **Anchor Peers Definition** 定义每个ORG的anchor peers的位置
    ````
    { "host": "peer0.org2.example.com", "port": 7051 }
    ````
* **Hashing Structure** Block里面的数据是一个字节数组的数组。Block的Hash被计算Merkle树。该参数指定了Merkle树的宽度。目前，这个值固定为4294967295，对应于Block级联字节数据的平坦散列。
    ````
    { "width": 4294967295 }
    ````
* **Hashing Algorithm** 用于计算编码到Block中散列值的算法。特别地，这会影响Block的Data hash和Previous block hash字段。注意，此字段当前只有一个有效值（SHA256），不应更改。
    ````
    { "name": "SHA256" }
    ````
* **Block Validation** 该策略指定了确认Block是否有效的签名要求. 默认情况下，它需要来自ordering org的某个成员的签名。
    ````
    {
        "type": 3,
        "value": {
            "rule": "ANY",
            "sub_policy": "Writers"
        }
    }
    ````
* **Orderer Address** Clients可以调用 Orderer Broadcast 和 Deliver功能的地址列表。Peers在这些地址中随机选择，并在它们之间检索获取Block失败信息。
    ````
    {
        "addresses": [
            "orderer.example.com:7050"
        ]
    }
    ````

正如我们通过添加ORG的artifacts和MSP信息来添加ORG一样，我们也可以通过反转这个过程来移除它们。
> **注意:** 一旦定义了````consensus type````并且网络已经被成功引导, 那么则不能通过````configuration update````来修改它.

另外, 还有一个重要的Channel Configuration（特别是对于V1.1）称之为 [Capability Requirements](https://linkto).

假设您想修改Block的BlockSize字段。

首先，为了方便引用JSON路径，我们将其定义为环境变量。
````export MAXBATCHSIZEPATH=".channel_group.groups.Orderer.values.BatchSize.value.max_message_count"````

接下来,查看该属性的值
````jq "$MAXBATCHSIZEPATH" config.json````

现在，让我们设置新的Block Batch的大小并显示新的值：
````
jq “$MAXBATCHSIZEPATH = 20” config.json > modified_config.json
jq “$MAXBATCHSIZEPATH” modified_config.json
````

一旦修改了JSON，就可以进行转换和提交。[添加Organiaztion](https://linkto)章节介绍了转换过程，下面让我们看看提交的过程。

## 获取必要的签名

一旦您成功生成了````protobuf````文件，就到了签名的时候了。要做到这一点，你需要先了解相关政策，无论你试图改变什么。

默认情况下，更改和签名对应如下：
* **A particular org**（例如，改变anchor peers）只需要该ORG的管理员签名。
* **The application** 需要大多数application organizations的管理员签名。
* **The orderer** 需要大多数ordering organizations的管理员签名（其中默认只有1个）。
* **The top level channel group** 需要大多数application organizations管理员和orderer organizations管理员的同意(签字)。

如果对Channel中的默认策略进行了更改，则需要相应地计算签名要求。

> **注意：** 您可以根据您的应用程序编写签名集合。一般来说，你可能总是收集比要求更多的签名。

获取这些签名的实际过程将取决于如何设置系统，但有两种主要实现方式。目前，Fabric命令行默认为“pass it along system”。也就是说，ORG的管理员会发起一个更新提案并将更新信息发送给需要签名的其他人（通常是另一个管理员）。这个管理员会（或不会）对其进行签名,然后将它传递给下一个管理员，以此类推，直到提交的配置有足够的签名。

这具有简单性的优点——当有足够的签名时，最后一个管理员可以简单地提交configuration transaction（在Fabric，````peer channel update````命令默认包含签名）。然而，这个过程只在较小的Channel中是可行的，因为“pass it along system”的方法可能是耗时的。

另一种选择是在一个Channel上向所有管理员提交更新，并等待足够的签名返回。然后可以将这些签名拼接在一起并提交。这使得创建配置更新的管理员（强迫他们处理每个签名者的文件）的生活更加困难，但对于正在开发Fabric管理应用程序的用户来说，这是推荐的工作流。


一旦配置被添加到账本中，一个良好的习惯是, 重新获取Fabric配置, 将其转换为JSON格式，并检查以确保所有的东西都被正确添加。















