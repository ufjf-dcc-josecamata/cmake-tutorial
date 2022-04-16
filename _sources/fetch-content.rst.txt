.. _fetch-content:


A08 - Manipulação de dependência automatizada com ``FetchContent``
===================================================================

.. questions:: Questão

   - Existe uma maneira de satisfazer automaticamente as dependências do nosso código?

.. objectives:: Objetivos

   - Aprenda como baixar suas dependências em tempo de configuração com ``FetchContent``.
   - Saiba como o conteúdo buscado pode ser usado de forma nativa em 
     seu sistema de compilação.


O CMake oferece **dois módulos** para satisfazer as dependências ausentes 
em tempo real: os módulos ``ExternalProject`` e ``FetchContent``.

Usando ``ExternalProject``
    - A etapa de download acontece em `tempo de construção do projeto
      <https://cmake.org/cmake/help/latest/module/ExternalProject.html>`_.
    - Você pode lidar com dependências que **não** usam o CMake.
    - Você precisa reescrever todo o seu sistema de compilação como um `superbuild
      <https://github.com/dev-cafe/cmake-cookbook/blob/master/chapter-08/README.md>`_.
Usando ``FetchContent``
    - A etapa de download acontece em `tempo de configuração do projeto
      <https://cmake.org/cmake/help/latest/module/FetchContent.html>`_.
    - Você só pode gerenciar dependências que usam CMake.
    - É uma alteração bem delimitada em um sistema de compilação CMake existente.


Ambos são mecanismos extremamente poderosos, mas você deve usá-los com cuidado. 
Muitas vezes, uma documentação abrangente será suficiente para ajudar os usuários 
a configurar seu ambiente para construir seu código com sucesso! Nesta atividade, 
discutiremos o módulo ``FetchContent``.


O modulo ``FetchContent`` 
---------------------------

Para buscar dependências dinamicamente no momento da configuração, 
você incluirá o módulo interno do CMake ``FetchContent``. Este módulo 
faz parte do CMake desde sua versão 3.11 e vem sendo aprimorado constantemente 
desde então.

Há duas etapas em um fluxo de trabalho baseado em ``FetchContent``:

#. **Declarando** o conteúdo a ser buscado com |FetchContent_Declare|. 
     Pode ser um tarball (local ou remoto), uma pasta local ou um 
     repositório de controle de versão (Git, SVN, etc.).

   .. signature:: |FetchContent_Declare|

      .. code-block:: cmake

         FetchContent_Declare(<name> <contentOptions>...)

      O comando aceita como ``<contentOptions>`` as mesmas opções que daria 
      para ``ExternalProject_Add`` nas etapas *download* e 
      *update/patch*. veja mais em https://cmake.org/cmake/help/latest/module/ExternalProject.html#command:externalproject_add.

      Para baixar o código de um repositório Git, você deve definir o seguinte:

      - ``GIT_REPOSITORY``, a localização do repositório.
      - ``GIT_TAG``, a versão (tag, branch, commit hash) para o check out.

      Considerando que para baixar um tarball você só precisa definir:

      - ``URL``, a localização online do tarball.

#. **Populando** o conteúdo com |FetchContent_MakeAvailable|. 
     Este comando *adiciona* os alvos declarados no conteúdo externo ao 
     seu sistema de compilação.

   .. signature:: |FetchContent_MakeAvailable|

      .. code-block:: cmake

         FetchContent_MakeAvailable( <name1> [<name2>...] )


   Como os alvos do projeto externo são adicionados ao seu próprio projeto, 
   você poderá usá-los da mesma maneira que faria ao obtê-los por meio de uma 
   chamada para |find_package|: você pode usar o conteúdo *found* e *fetched* 
   na mesma maneira. Se você precisa definir opções para criar o projeto externo, 
   você as definirá como variáveis CMake *antes* de chamar 
   |FetchContent_MakeAvailable|.

Testes unitários com Catch2
+++++++++++++++++++++++++++++

O teste de unidade é uma técnica valiosa em engenharia de software: 
ele pode ajudar a identificar regressões funcionais com um nível muito fino de 
controle, pois cada teste de unidade destina-se a exercitar componentes 
isolados em sua base de código. Equipar sua base de código com 
integração *e* testes de unidade é uma prática muito boa.

Existem muitas estruturas de teste de unidade para a linguagem C++. 
Cada um deles enfatiza uma abordagem ligeiramente diferente para 
testes de unidade e vem com suas próprias peculiaridades na 
configuração e uso.

Nesta atividade, mostraremos como usar `Catch2
<https://github.com/catchorg/Catch2>`_ uma estrutura de teste de unidade 
muito popular que enfatiza um fluxo de trabalho de desenvolvimento orientado 
a testes. Catch2 é distribuído como um arquivo de cabeçalho único, que é uma 
de suas características mais atraentes: pode ser facilmente incluído em 
qualquer projeto. Em vez de baixar o arquivo de cabeçalho e adicioná-lo à 
nossa base de código, podemos usar ``FetchContent`` para satisfazer essa 
dependência para nós quando necessário.


.. exercise:: Exercício 26: Catch2 recarregado

   Queremos usar a estrutura de teste de unidade Catch2 em nosso código. 
   Neste exercício, faremos o download do projeto Catch2 no momento da 
   configuração de seu `repositório GitHub <https://github.com/catchorg/Catch2>`_.

   O projeto base está em ``source/code/day-2/26_more-catch2``.

   #. Crie um projeto C++.
   #. Defina o padrão C++ para C++14. Catch2 também funcionará com C++11.
   #. Crie uma biblioteca a partir do arquivo fonte ``sum_integers.cpp``.
   #. Vincule a biblioteca em um executável ``sum_up``.
   #. Inclua o módulo ``FetchContent`` e declare o conteúdo ``Catch2``. 
      Queremos baixar a tag ``v2.13.4`` do `repositório oficial do Git <https://github.com/catchorg/Catch2>`_.
   #. Faça o conteúdo ``Catch2`` disponível.
   #. Crie o executável ``cpp_test``.
   #. Habilite o teste e adicione um teste. 
      Você terá que verificar como chamar um executável Catch2 na `documentação <https://github.com/catchorg/Catch2/blob/v2.x/docs/command-line.md#specifying-which-tests-to- executar>`_.
   #. Tente rodar seus testes.

   - Que diferenças você observa na etapa de configuração?
   - O que acontece se você esquecer de emitir o comando |FetchContent_MakeAvailable|?
   - Quais alvos são construídos no projeto? Quais são do Catch2?  Você pode usar o seguinte comando para obter uma lista de todos os destinos disponíveis:

     .. code-block:: bash

        $ cmake --build build --target help

   Uma solução funcional está na subpasta ``solution``.

.. warning:: Atençao

   ``FetchContent`` é um módulo poderoso em sua caixa de ferramentas CMake. 
   **Cuidado!** Satisfazer *todas* as dependências do seu código dessa forma 
   fará com que a duração dos estágios de configuração e construção aumentem.


.. keypoints:: Resumo

   - CMake permite que você satisfaça dependências *on-the-fly*.
   - Você pode fazer isso em tempo de compilação com ``ExternalProject``, mas você precisa adotar um framework superbuild.
   - No momento da configuração, você pode usar o módulo ``FetchContent``: ele só pode ser aplicado com dependências que também usam CMake.
