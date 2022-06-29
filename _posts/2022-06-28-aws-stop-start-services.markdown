---
layout: post
title:  "AWS: Stop/Start services ECS"
date:   2022-06-28 20:00:00 +0000
categories: jekyll update
---

Olá, tudo bem? 
Esse é o meu primeiro post no site, o objetivo sempre será o compartilhamento de conhecimento, dicas e experiências sobre Cloud e DevOps.

Então vamos lá, há um certo tempo, me deparei com a necessidade de ligar e desligar o ambiente não produtivo em ECS conforme jornada das fábricas de software.
No primeiro momento, utilizei uma imagem docker amazon/aws-cli [aws-cli] em conjunto com jobs agendados no Ansible AWX [ansible-awx], mas o job falhava a cada alteração de serviços no cluster. Isso começou a ficar oneroso, pois era necessário ajustar o template em diversos jobs.

Após validar algumas alternativas, acabei aprofundando um pouco mais no AWS SDK para Python [aws-sdk]. Com o script em Python (Boto3), houve uma redução significativa no tempo de execução do script, além de eliminar a necessidade de ajuste no template a cada alteração de serviços. 
Vou compartilhar com vocês o script que foi grande parte da solução do problema. 

* Configuração do ambiente 

É necessário a instalação do Boto3 no Python e configurar as credenciais de autenticação na AWS, segue o link com o passo a passo [aws-sdk-config].
O ideal é executar o script via EC2 ou Lambda com IAM roles, caso não seja possível, atente-se a configuração do profile por AWS CLI. Na etapa abaixo, estamos apontando o perfil default para autenticação.

{% highlight ruby %}
boto3.setup_default_session(profile_name='default')
{% endhighlight %}

* Listar/Desligar/Ligar serviços do cluster ECS

Para reaproveitar o script, vamos executá-lo com 03 parâmetros, nome do cluster, região e 0 para desligar ou 1 para ligar. Siga essa ordem para não falhar a execução.

{% highlight ruby %}
python3 script.py nome_do_cluster_ecs região 0(desligar) ou 1(ligar) 
{% endhighlight %}

    Desligando os serviços
{% highlight ruby %}
python3 script.py app-dev us-east-1 0
{% endhighlight %}

    Ligando os serviços
{% highlight ruby %}
python3 script.py app-dev us-east-1 1
{% endhighlight %}

Ao final da execução, deverá aparecer a mensagem "Sucess", indicando que o script foi executado com sucesso.

* Script Python

Segue o script python que tanto estou falando;

{% highlight ruby %}

import boto3
import sys
from botocore.exceptions import ClientError

boto3.setup_default_session(profile_name='default')

client = boto3.client('ecs', region_name=sys.argv[2])

try:
    response_list = client.list_services(
        cluster=sys.argv[1],
        maxResults=100
    )

except ClientError as error:
    print(error)

try:
    for listservice in response_list['serviceArns']:
        client.update_service(
            cluster=sys.argv[1],
            service=listservice,
            desiredCount=sys.argv[3]
        )
except ClientError as error:
    print(error)

print('Success')

{% endhighlight %}

---

É isso aí, nos vemos em um proximo post.

Grande abraço.

**Serrano**

[aws-cli]: https://hub.docker.com/r/amazon/aws-cli
[ansible-awx]:   https://github.com/ansible/awx
[aws-sdk]: https://aws.amazon.com/pt/sdk-for-python/
[aws-sdk-config]: https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#installation
