# Projeto-M2C

# Apresentação da Demanda
O Projeto M2C vem do direcional executivo e tem por objetivo mover toda a aplicação do Datacenter on-premises para a Cloud com objetivo na descontinuidade dos Datacenter próprios.
Futuramente, toda a parte de Storage também será movida para a Cloud.

## Macro Escopo
Atualmente o parque de servidores de nossos Datacenter on-premises estão em processo de desativação devido a falta de suporte e as aplicações estão sendo migradas para a Nuvem.
A arquitetura de integração dos sistemas legado está baseado em uma arquitetura já obsoleta e sem suporte do fornecedor com o custo muito alto de manutenção e spare parts comprometido. 
O objetivo dessa demanda é prover a atualização tecnológica do barramento de serviços com soluções em container na cloud, com desenvolvimento LowCode para uma maior produtividade e seguir a metodologia DevOps.
![image](https://github.com/user-attachments/assets/6faa6e49-735f-4409-b9b9-337135fd3472)


## Premissas 
Migração do ambiente on-premises deve ser realizada de forma faseada garantindo que não teremos indisponiblidade de forma que o ambiente seja impactado assegurundo a eficiência e performance na comunicação dos sistemas.

Garantir a escalabilidade e alta disponibilidade "HA" do ambiente.





## Risco e Restrições
### Migrar os serviços do Legado para a arquitetura de integração meta da empresa
A implantação do novo ambiente será realizada através de Janela de Manutenção com GMUD aprovada dentro do Comitê Executivo de Mudanças que serão planejadas por 2 servidores cada uma delas em horário de baixo tráfego e em final de semana.

Mediante reunião realizada entre Gerente de Projetos, Lider Técnico, Capacity, Segurança, Integração e Arquitetura, todo o alinhamento necessário foi realizado.

Nesse formato acima, todos os envolvidos no projeto entende que a infraestrutura não sofrerá impacto em sua solução atual durante a Janela de Manutenção.
 




## Volumetria:
A volumetria atual não sofrerá alterações, mas necessitará da criação de uma infraestrutura paralela para conviver com a infraestrutura atual, até que todos os requisitos funcionais tenham sido migrados, testados e homologados dentro do novo ambiente.

Para quantificar a quantidade de transações que entram no Barramento de serviços, temos que considerar a coluna "Transações Entrada" na tabela. Ou seja, uma aplicação consumindo o serviço.
A coluna "Orquestrações Internas" representam todo o fluxo executado pelos serviços consumindo as camadas internas do serviço xpto.

![image](https://github.com/user-attachments/assets/1e261991-6ade-48a1-b06d-9fd9f1357f74)

![image](https://github.com/user-attachments/assets/3e8d1e8e-6937-4b22-a488-72c2588134ff)










