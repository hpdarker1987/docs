---
title: Сведения о центрах сертификации SSH
intro: 'С помощью центра сертификации SSH ваша учетная запись организации или предприятия может предоставлять сертификаты SSH, которые участники могут использовать для доступа к ресурсам посредством Git.'
redirect_from:
  - /articles/about-ssh-certificate-authorities
  - /github/setting-up-and-managing-organizations-and-teams/about-ssh-certificate-authorities
versions:
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - Organizations
  - Teams
shortTitle: SSH certificate authorities
ms.openlocfilehash: e7f114ddbfabe9b386fce61cedd22fa669ff542c
ms.sourcegitcommit: d697e0ea10dc076fd62ce73c28a2b59771174ce8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/20/2022
ms.locfileid: '148094452'
---
## Сведения о центрах сертификации SSH

Сертификат SSH используется для подписания одного ключа SSH другим ключом SSH. Если вы предоставляете участникам организации подписанные сертификаты SSH с помощью центра сертификации (ЦС) SSH, этот центр можно добавить в вашу корпоративную учетную запись или организацию. Это позволит участникам организации использовать свои сертификаты для доступа к ресурсам организации. 

{% data reusables.organizations.ssh-ca-ghec-only %}

ЦС SSH, добавленный в организацию или корпоративную учетную запись, можно использовать для подписывания сертификатов клиента SSH для участников организации. Участники организации могут использовать подписанные сертификаты для получения доступа к репозиториям только вашей организации с помощью Git. Использование участниками сертификатов SSH для доступа к ресурсам организации можно сделать обязательным. Дополнительные сведения см. в разделах [Управление центрами сертификации SSH в вашей организации](/articles/managing-your-organizations-ssh-certificate-authorities) и [Применение политик для параметров безопасности в вашем предприятии](/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-security-settings-in-your-enterprise#managing-ssh-certificate-authorities-for-your-enterprise).

Например, вы можете создать внутреннюю систему, которая будет каждое утро выдавать разработчикам новый сертификат. Каждый разработчик сможет использовать свой ежедневный сертификат для работы с репозиториями вашей организации в {% data variables.product.product_name %}. В конце дня срок действия сертификата может автоматически истекать, благодаря чему ваши репозитории будут защищены от его последующей компрометации.

{% ifversion ghec %} Участники организации могут использовать подписанные сертификаты для проверки подлинности, даже если применяется единый вход SAML. Если вы не сделаете сертификаты SSH обязательным, члены организации могут продолжать использовать другие средства проверки подлинности для доступа к ресурсам организации с помощью Git, включая имя пользователя и пароль, {% данных variables.product.pat_generic %}s и собственные ключи SSH.
{% endif %}

Участники не смогут использовать свои сертификаты для доступа к вилкам репозиториев, которые принадлежат их личным учетным записям. 

## Сведения о URL-адресах SSH с сертификатами SSH

Чтобы исключить ошибки проверки подлинности при выполнении операций Git по протоколу SSH в тех случаях, когда использование сертификатов SSH в организации обязательно, участникам организации необходимо использовать специальный URL-адрес, содержащий идентификатор организации. Это позволит клиенту и серверу упростить согласование используемого для проверки подлинности ключа на компьютере участника. Если участник использует обычный URL-адрес, начинающийся с `git@github.com`, клиент SSH может предложить неправильный ключ, что приведет к сбою операции.

Этот URL-адрес любой пользователь с доступом на чтение к репозиторию может найти в раскрывающемся меню **Код** на главной странице репозитория, щелкнув **Использовать SSH**.

Если использование сертификатов SSH в вашей организации не обязательно, участники могут продолжать использовать собственные ключи SSH или другие способы проверки подлинности. В таком случае подойдет как специальный URL-адрес, так и обычный URL-адрес, начинающийся с `git@github.com`.

## Выдача сертификатов

При выдаче каждого сертификата необходимо включить расширение, указывающее, для какого пользователя {% data variables.product.product_name %} используется сертификат. Например, можно использовать команду OpenSSH `ssh-keygen`, заменив _KEY-IDENTITY_ на удостоверение вашего ключа, а _USERNAME_ — на имя пользователя {% data variables.product.product_name %}. Создаваемый сертификат будет давать право действовать от имени этого пользователя при работе с любыми ресурсами вашей организации. Прежде чем выдавать сертификат, проверьте личность пользователя.

{% note %}

**Примечание.** Чтобы использовать эти команды, необходимо обновить до OpenSSH 7.6 или более поздней версии.

{% endnote %}

```shell
$ ssh-keygen -s ./ca-key -V '+1d' -I KEY-IDENTITY -O extension:login@{% data variables.product.product_url %}=USERNAME ./user-key.pub
```

{% warning %}

**Предупреждение**. После подписания и выдачи сертификата отозвать его нельзя. Обязательно настройте время существования сертификата с помощью флага -`V`, иначе он будет бессрочным.

{% endwarning %}

Чтобы выдать сертификат для пользователя, который использует SSH для доступа к нескольким продуктам {% data variables.product.company_short %}, можно включить два расширения входа и указать имя пользователя для каждого продукта. Например, следующая команда выдает сертификат для _USERNAME-1_ для учетной записи пользователя в {% data variables.product.prodname_ghe_cloud %} и для _USERNAME-2_ для учетной записи пользователя в {% data variables.product.prodname_ghe_managed %} или {% data variables.product.prodname_ghe_server %} на узле _HOSTNAME_.

```shell
$ ssh-keygen -s ./ca-key -V '+1d' -I KEY-IDENTITY -O extension:login@github.com=USERNAME-1 extension:login@HOSTNAME=USERNAME-2 ./user-key.pub
```

С помощью расширения `source-address` вы можете ограничить список IP-адресов, с которых участник организации может получать доступ к ее ресурсам. Это расширение принимает отдельные IP-адреса или диапазоны IP-адресов в нотации CIDR. Вы можете указать несколько адресов или диапазонов, разделяя значения запятыми. Дополнительные сведения см. в статье [Бесклассовая адресация](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation) в Википедии.

```shell
$ ssh-keygen -s ./ca-key -V '+1d' -I KEY-IDENTITY -O extension:login@{% data variables.product.product_url %}=USERNAME -O source-address=COMMA-SEPARATED-LIST-OF-IP-ADDRESSES-OR-RANGES ./user-key.pub
```
