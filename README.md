# phishing-url-detector

**Detector de URLs de phishing** 
-- -- -- -- -- -- -- -- -- -- --
*Classificador de URLs de phishing em Python (scikit-learn). O projeto compara três abordagens e analisa onde o melhor modelo acerta, onde erra e por quê.*


**PROBLEMA**
-- -- -- -- --
Phishing é um golpe em que o atacante imita um site legítimo (banco, e-mail, PayPal) para roubar credenciais. Filtros de navegador e de e-mail tentam bloquear esses links antes que o usuário clique. Aqui o objetivo é decidir, olhando só o texto da URL, se ela é phishing ou legítima.

Porém, é importante considerar que em segurança, os dois erros não custam igual:
• *Falso negativo (phishing que passa)*: o usuário pode ser enganado. É o erro mais caro
• *Falso positivo (site legítimo bloqueado)*: causa incômodo.

Por isso o recall da classe phishing é a métrica mais importante, e acompanhada de precisão e F1. A acurácia sozinha engana, já que só ~22% das URLs são phishing.

**DADOS**
-- -- -- --
• *Fonte*: dataset Phishing Site URLs do Kaggle (link do dataset), com duas colunas: URL e Label (good ou bad).
• *Original*: 549.346 linhas. Depois de remover URLs duplicadas: 507.195 (42.151 duplicatas removidas, concentradas na classe phishing). A remoção foi feita antes da divisão treino/teste, para que a mesma URL não apareça nos dois lados.
• *Proporção após a limpeza*: 77,5% legítimas e 22,5% phishing.
• O CSV não está neste repositório. Baixe do Kaggle e coloque na raiz do projeto.

**MÉTODO**
-- -- -- --
1. *Limpeza*: remoção de duplicatas e rótulo numérico (bad = 1 = phishing, good = 0).
2. *Divisão*: 80% treino e 20% teste (101.439 URLs de teste), estratificada pela classe, random_state=42.
3. *Três modelos, todos com class_weight="balanced" por causa do desbalanceamento*:
    • Regressão logística com 17 características extraídas à mão (padronizadas com StandardScaler).
    • Random forest (100 árvores) com as mesmas 17 características.
    • TF-IDF de n-gramas de caracteres (3 a 5) + regressão logística, que usa o texto bruto da URL e aprende sozinho quais pedaços indicam phishing.

Características manuais (17): tamanho da URL, do domínio e do caminho; contagem de pontos, hífens, barras, dígitos, @, ?, = e %; número de subdomínios; domínio em formato de IP; contagem de palavras suspeitas (login, verify, secure, account, update, confirm, signin, bank, paypal, webscr, ebay); terminação em .php ou .htm(l); entropia da string.

**RESULTADOS**
-- -- -- --
Avaliação no conjunto de teste, classe positiva = phishing:

|Modelo	                                 |          |Precisão|	 |Recall|	|F1   |
-----------------------------------------------------------------------------------
|Regressão logística (17 características)|	        |0,597	 |   |0,635|	|0,615|
-----------------------------------------------------------------------------------
|Random forest (17 características)|	            |0,802	 |   |0,832|	|0,817|
-----------------------------------------------------------------------------------
|TF-IDF de caracteres + regressão logística|	    |0,928	 |   |0,953|	|0,940|
-----------------------------------------------------------------------------------

Pontos principais:
    • Características desenhadas à mão levaram a um F1 de 0,82 com a random forest. A regressão logística nelas ficou em 0,62, pois só soma efeitos isolados e não captura combinações (por exemplo, URL longa e .php e palavra suspeita).
    • Deixar o modelo aprender padrões de caracteres subiu o F1 para 0,94, com recall de 0,95: cerca de 95% dos phishings do teste foram detectados.
    • Na random forest, 3.845 dos 22.860 phishings do teste passaram como legítimos (falsos negativos), contra 4.700 URLs legítimas bloqueadas (falsos positivos).

**O QUE O MODELO APRENDEU**
-- -- -- -- -- -- -- -- -- --
Os pedaços de URL com mais peso para phishing no melhor modelo incluem login, /log, pay, aol, mail, .php, .exe, /wp-, /js e /images. São padrões coerentes com páginas falsas de login e com sites WordPress invadidos usados para hospedar o golpe.

Já os pedaços com mais peso para "legítima" incluem .edu/, .org/, .ca/, wiki, film, id= e .htm. Isso mostra que o modelo também aprende de onde os dados vieram, e não só o que é phishing (veja Limitações).

**LIMITAÇÕES**
-- -- -- -- --
    • Viés de fonte. Os phishings e as URLs legítimas do dataset parecem vir de origens e épocas diferentes. O modelo usa terminações de domínio e estilo de caminho como atalho. O F1 de 0,94 vale para este dataset e provavelmente cai em URLs atuais. Não foi feito teste em um segundo dataset.
    • Falsos negativos típicos. Phishing hospedado em sites legítimos invadidos (por exemplo, domínios .edu ou .org com um caminho aleatório) e domínios sem caminho (dominio.com/) passam despercebidos, porque a URL sozinha não dá sinal suficiente. Detectá-los exigiria olhar o conteúdo da página ou o histórico do domínio.
    • Ruído nos dados. 397 URLs (0,08%) têm caracteres não-ASCII, 93% delas rotuladas como legítimas, incluindo entradas que parecem corrompidas. O impacto no resultado é desprezível, mas indica ruído na fonte.
    • Sem esquema na URL. As URLs vêm sem http:// ou https://, então não foi possível usar o protocolo como característica.
    • Avaliação simples. Uma única divisão treino/teste, sem validação cruzada e sem ajuste de hiperparâmetros.

**PRÓXIMOS PASSOS**
-- -- -- -- -- -- --
    • Testar o modelo em um dataset de outra fonte para medir a generalização.
    • Validação cruzada e ajuste de hiperparâmetros.
    • Combinar as características manuais com o TF-IDF.
    • Adicionar características de domínio (idade do domínio, WHOIS) e de conteúdo da página.

**TECNOLOGIAS**
-- -- -- -- -- --
Python, pandas, scikit-learn, matplotlib, Jupyter