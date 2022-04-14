.. _dependencies:


Encontrando e usando dependências
===================================

.. questions::

   - Como posso usar o CMake para detectar e usar as dependências do meu projeto?

.. objectives::

   - Aprender como usar |find_package|.
   - Saiba quais outras alternativas de detecção existem.

A grande maioria dos projetos de software não acontece no vácuo: 
eles terão dependências de frameworks e bibliotecas existentes. 
Uma boa documentação instruirá seus usuários a garantir que eles 
sejam satisfeitos em seu ambiente de programação. 
O sistema de compilação é o local apropriado para verificar se 
essas pré-condições são atendidas e se seu projeto pode ser construído 
corretamente. Neste episódio, mostraremos alguns exemplos de como 
detectar e usar dependências em seu sistema de compilação CMake.

Encontrando dependências
--------------------------

O CMake oferece uma família de comandos para encontrar artefatos instalados em seu sistema:

- |find_file| para recuperar o caminho completo para um arquivo.
- |find_library| para encontrar uma biblioteca, compartilhada ou estática.
- |find_package| para encontrar e carregar configurações de um projeto externo.
- |find_path| para encontrar o diretório que contém um arquivo.
- |find_program| para encontrar um executável.

A principal ferramenta para a descoberta de dependência é |find_package|, 
que cobrirá suas necessidades em quase todos os casos de uso.

.. signature:: |find_package|

   .. code-block:: cmake

      find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [NO_POLICY_SCOPE])

   Este comando tentará encontrar o pacote com o nome ``<PackageName>`` pesquisando 
   em várias `pastas predefinidas <https://cmake.org/cmake/help/latest/command/find_package.html?highlight=find_package#search-procedure>`_.
   É possível solicitar uma versão mínima ou exata. Se ``REQUIRED`` for fornecido, uma pesquisa 
   com falha acionará um erro fatal. As regras para a busca são obtidas em módulos chamados 
   ``Find<PackageName>.cmake``.
   Os pacotes também podem ter *componentes* e você pode pedir para detectar apenas alguns deles. 

você deve **somente** usar os outros comandos da família ``find_`` 
em circunstâncias muito especiais e restritas. Por quê?

1. Para uma grande variedade de dependências comuns, os módulos ``Find<PackageName>.cmake`` enviados 
   com o CMake funcionam perfeitamente e são mantidos pelos desenvolvedores do CMake. 
   Isso elimina o fardo de programar seus próprios modulos de detecção de dependência.
2. |find_package| irá configurar **alvos importados**: alvos definidos *fora* do
   seu projeto que podem ser usados com seus próprios alvos. 
   As propriedades nos destinos importados definem *requisitos de uso* para as 
   dependências. Um comando como:

   .. code-block:: cmake

      target_link_libraries(your-target
        PUBLIC
          imported-target
        )


   definirá *flags* de compilador, definições, diretórios de inclusão e bibliotecas de links de ``imported-target`` para ``your-target`` *e* para todos os outros destinos em seu projeto 
   que usarão ``your-target``.


Esses dois pontos simplificam **enormemente** a
detecção de dependência e uso consistente em um projeto de várias pastas.


Usando ``find_package``
++++++++++++++++++++++++

Ao tentar a detecção de dependência com |find_package|, você deve certificar-se de que:

- Um modulo ``Find<PackageName>.cmake`` exista,
- Quais componentes, se houver, ele fornece e
- Quais destinos importados ele configurará.

Uma lista completa de ``Find<PackageName>.cmake`` pode ser encontrada na interface de linha de comando:
.. code-block:: bash

   $ cmake --help-module-list | grep "Find"

.. typealong:: Usando OpenMP.

   Queremos compilar o seguinte código de exemplo do OpenMP: [#omp]_


   .. literalinclude:: code/day-2/22_taskloop/solution/taskloop.cpp
      :language: c++

   Observe o uso da construção ``taskloop``, que foi introduzida no OpenMP 4.5: 
   precisamos ter certeza de que nosso compilador C++ é compatível com *pelo menos* 
   essa versão do padrão.

   Da documentação do módulo ``FindOpenMP.cmake``:

   .. code-block:: bash

      $ cmake --help-module FindOpenMP | less

   descobrimos que o módulo fornece os componentes ``C``, ``CXX`` e ``Fortran`` 
   e que o destino ``OpenMP::OpenMP_CXX`` será fornecido, 
   se a detecção for bem sucedida. Assim, fazemos o seguinte:

   .. code-block:: cmake

      find_package(OpenMP 4.5 REQUIRED COMPONENTS CXX)

      target_link_libraries(task-loop PRIVATE OpenMP::OpenMP_CXX)

   Podemos configurar e construir detalhadamente. [#verbose]_
   Observe que os sinalizadores do compilador, diretórios de inclusão e 
   bibliotecas de links são resolvidos corretamente pelo CMake.
   
   Você pode encontrar o exemplo completo de trabalho em ``source/code/day-2/22_taskloop/solution``.

.. exercise:: Exercício 23: Usando MPI

   Neste exercício, você tentará compilar um programa "Hello, world" 
   que usa a interface de passagem de mensagens (MPI).

   1. Verifique se existe um módulo ``FindMPI.cmake`` na biblioteca de módulos embutida.
   2. Familiarize-se com seus componentes e as variáveis e destinos 
      importados que ele define.

   .. tabs::

      .. tab:: C++

         O projeto base está em ``source/code/day-2/23_mpi-cxx``.

         #. Compile o arquivo de origem em um executável.
         #. Vincule para o destino importado MPI.
         #. Invoque uma compilação detalhada e observe como o CMake compila e vincula.

         Um exemplo funcional está na subpasta ``solution``.

      .. tab:: Fortran

          O projeto base está em ``content/code/day-2/23_mpi-f``.

         #. Compile o arquivo de origem em um executável.
         #. Vincule para o destino importado MPI.
         #. Invoque uma compilação detalhada e observe como o CMake compila e vincula

          Um exemplo funcional está na subpasta ``solution``.


Alternativas: scripts ``Config`` e ``pkg-config``
+++++++++++++++++++++++++++++++++++++++++++++++++++

O que fazer quando não há um módulo ``Find<PackageName>.cmake`` embutido para um pacote do qual você depende?
Os desenvolvedores de pacotes podem já estar preparados para ajudá-lo:

- Eles disponibilizam o arquivo específico do CMake ``<PackageName>Config.cmake`` que descreve 
  como o destino importado deve ser usada pelo seu pacote. Neste caso, você precisa 
  apontar o CMake para a pasta que contém o arquivo ``Config`` usando a variável 
  especial ``<PackageName>_DIR``:

  .. code-block:: bash

     $ cmake -S. -Bbuild -D<PackageName>_DIR=/folder/containing/<PackageName>Config.cmake

- Eles incluem um arquivo ``.pc``, que, em plataformas do tipo Unix, pode ser detectado 
  com o utilitário ``pkg-config``. Você pode então aproveitar o ``pkg-config`` 
  através do CMake:

  .. code-block:: cmake

     # find pkg-config
     find_package(PkgConfig REQUIRED)
     # ask pkg-config to find the UUID library and prepare an imported target
     pkg_search_module(UUID REQUIRED uuid IMPORTED_TARGET)
     # use the imported target
     if(TARGET PkgConfig::UUID)
       message(STATUS "Found libuuid")
     endif()

Esta foi a estratégia adotada em :ref:`probing` ao testar o uso da biblioteca UUID.

.. keypoints::

   - O CMake possui um rico ecossistema de módulos para encontrar dependências de software. Eles são chamados de ``Find<package>.cmake``.
   - Os módulos ``Find<package>.cmake`` são usados através de ``find_package(<package>)``.
   - Você também pode usar a ferramenta clássica do Unix ``pkg-config`` 
      para encontrar dependências de software, mas isso não é tão robusto quanto os módulos ``Find<package>`` nativos do CMake.



.. rubric:: Footnotes

.. [#omp]

   Exemplo adaptado da página 85 em `OpenMP 4.5 examples
   <http://www.openmp.org/wp-content/uploads/openmp-examples-4.5.0.pdf>`_.

.. [#verbose]

   A maneira de acionar uma compilação detalhada depende da ferramenta de compilação nativa que você está usando.
   Para Makefiles Unix:

   .. code-block:: bash

      $ cmake --build build -- VERBOSE=1

   Para Ninja:

   .. code-block:: bash

      $ cmake --build build -- -v
