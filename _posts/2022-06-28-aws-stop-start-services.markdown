---
layout: post
title:  "AWS: Stop/Start services"
date:   2022-06-28 20:00:00 +0000
categories: jekyll update
---

Olá pessoal, tudo bem? 
Esse é o meu primeiro post no site, o objetivo sempre será o compartilhamento de conhecimento, dicas e experiências sobre Cloud e DevOps.

Então vamos lá, há um certo tempo, me deparei com a necessidade de ligar e desligar o ambiente não produtivo em ECS conforme jornada das fábricas de software.
No primeiro momento, utilizei uma imagem docker amazon/aws-cli [aws-cli] em conjunto com jobs agendados no Ansible AWX [ansible-awx], mas o job falhava a cada alteração de serviços no cluster. Isso começou a ficar oneroso, pois era necessário ajustar o template em diversos jobs.

Após validar algumas alternativas, acabei aprofundando um pouco mais no AWS SDK para Python [aws-sdk]. Com o script em Python (Boto3), houve uma redução significativa no tempo de execução do script, além de eliminar a necessidade de ajuste no template a cada alteração de serviços. 
Vou compartilhar com vocês o script que foi grande parte da solução do problema. 

1. Configuração do ambiente 

É necessário a instalação do Boto3 no Python e configurar as credenciais de autenticação na AWS, o passo a passo esta nesse link [aws-sdk-config].
O ideal é executar o script via EC2 ou Lambda com IAM roles, caso não seja possível, atente-se a configuração do profile por AWS CLI. Na etapa abaixo, estamos apontando o perfil default para autenticação.

{% highlight ruby %}
boto3.setup_default_session(profile_name='default')
{% endhighlight %}

2. Listar/Desligar/Ligar serviços do cluster ECS

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


[aws-cli]: https://hub.docker.com/r/amazon/aws-cli
[ansible-awx]:   https://github.com/ansible/awx
[aws-sdk]: https://aws.amazon.com/pt/sdk-for-python/
[aws-sdk-config]: https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#installation
