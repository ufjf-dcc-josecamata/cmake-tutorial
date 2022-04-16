.. _targets:


A06 -Sistemas de compilação baseados em alvos com CMake
=========================================================

.. questions:: Questões

   - Como podemos lidar com projetos mais complexos com o CMake?
   - O que exatamente são **alvos (targets)** na linguagem específica de domínio (DSL) do CMake?

.. objectives:: Objetivos

   - Aprenda que os elementos básicos no CMake *não* são  variáveis, *mas* alvos.
   - Saiba mais sobre as propriedades dos alvos e como usá-los.
   - Aprenda a usar *níveis de visibilidade* para expressar dependências entre alvos.
   - Saiba como trabalhar com projetos que abrangem várias pastas.
   - Saiba como lidar com multiplos alvos em um projeto.

Projetos do mundo real exigem mais do que compilar alguns arquivos fontes 
em executáveis e/ou bibliotecas. Na grande maioria dos casos, você se deparará 
com projetos que compreendem centenas de arquivos fontes espalhados em uma estrutura 
complexa. O uso do CMake ajuda a manter a complexidade do sistema de compilação
sob controle.

É tudo sobre alvos e propriedades
-------------------------------------

Com o advento do CMake 3.0, também conhecido como **Modern CMake**, houve uma mudança 
significativa na forma como a linguagem específica de domínio (DSL) do 
CMake é estruturada. Em vez de depender de **variáveis** para transmitir 
informações em um projeto, devemos passar a usar **alvos** e **propriedades**.

Alvos (Targets)
++++++++++++++++

Um alvo é declarado por |add_executable| ou |add_library|: assim, 
em termos gerais, um destino mapeia para um artefato de construção 
no projeto. [#custom_targets]_ 
Qualquer destino tem uma coleção de **propriedades**, que definem *como* o 
artefato de compilação deve ser produzido **e** *como* ele deve ser usado 
por outros destinos dependentes no projeto.

.. figure:: img/target.svg
   :align: center

   Um alvo é o elemento básico no CMake DSL. Cada destino possui *propriedades*, 
   que podem ser lidas com |get_target_property| e modificado 
   com |set_target_properties|. Opções de compilação, definições, 
   diretórios de inclusão, arquivos de origem, bibliotecas de links e 
   opções de links são propriedades dos alvos.

É muito mais robusto usar alvos e propriedades do que usar variáveis.
Dado um alvo ``tgtA``, podemos invocar um comando na família ``target_*`` como:

.. code-block:: cmake

   target_link_libraries(tgtA
     PRIVATE tgtB
     INTERFACE tgtC
     PUBLIC tgtD
     )

O uso dos níveis de visibilidade podem ser os seguintes:

- ``PRIVATE``. A propriedade só será usada para construir o alvo dado 
  como primeiro argumento. Em nosso pseudo-código, ``tgtB`` será usado 
  apenas para construir ``tgtA`` mas não será propagado como uma 
  dependência para outros alvos consumindo ``tgtA``.
- ``INTERFACE``. A propriedade será usada apenas para construir destinos que 
   consumam o alvo fornecido como primeiro argumento. Em nosso pseudo-código, 
   o ``tgtC`` só será propagado como uma dependência para outros alvos que 
   consomem ``tgtA``.
- ``PUBLIC``. A propriedade será usada **em ambos** para construir o destino 
   fornecido como o primeiro argumento **e** os destinos que o consomem. 
   Em nosso pseudo-código, ``tgtD`` será usado para construir ``tgtA`` e 
   será propagado como uma dependência para qualquer outro alvo 
   que consuma ``tgtA``.


.. figure:: img/target_inheritance.svg
   :align: center

   As propriedades dos alvos têm **níveis de visibilidade** que determinam 
   como o CMake deve propagá-las entre alvos interdependentes.

Os cinco comandos mais usados para lidar com alvos são:

.. signature:: |target_sources|

   .. code-block:: cmake

      target_sources(<target>
        <INTERFACE|PUBLIC|PRIVATE> [items1...]
        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

   Use-o para especificar quais arquivos fontes devem ser usados ao 
   compilar um alvo.


.. signature:: |target_compile_options|

   .. code-block:: cmake

      target_compile_options(<target> [BEFORE]
        <INTERFACE|PUBLIC|PRIVATE> [items1...]
        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

   Use-o para especificar quais *flags* de compilador devem ser usados.

.. signature:: |target_compile_definitions|

   .. code-block:: cmake

      target_compile_definitions(<target>
        <INTERFACE|PUBLIC|PRIVATE> [items1...]
        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

   Use-o para especificar quais definições do compilador devem ser usadas.

.. signature:: |target_include_directories|

   .. code-block:: cmake

      target_include_directories(<target> [SYSTEM] [BEFORE]
        <INTERFACE|PUBLIC|PRIVATE> [items1...]
        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

  Use-o para especificar quais diretórios conterão arquivos de 
  cabeçalho (para C/C++) e de módulo (para Fortran).

.. signature:: |target_link_libraries|

   .. code-block:: cmake

      target_link_libraries(<target>
        <PRIVATE|PUBLIC|INTERFACE> <item>...
        [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)

   Use-o para especificar quais bibliotecas devem ser vinculadas ao alvo atual.

Existem comandos adicionais na família ``target_*``. Para ver quais são eles, faça:

.. code-block:: bash

   $ cmake --help-command-link | grep "^target_"

Propriedades
+++++++++++++

Até agora vimos que você pode definir propriedades em alvos, 
mas também em testes (veja :ref:`hello-ctest`).
O CMake permite definir propriedades em vários níveis diferentes 
de visibilidade em todo o projeto:

- **Escopo Global**. Elas são equivalentes às variáveis definidas na raiz ``CMakeLists.txt``. Seu uso é, no entanto, 
  mais poderoso, pois eles podem ser definidos a partir 
  de *qualquer* folha ``CMakeLists.txt``.
- **Escopo do diretório**. Elas são equivalentes a variáveis definidas em uma determinada folha ``CMakeLists.txt``.
- **Target**. Essas são as propriedades definidas nos alvos que discutimos acima.
- **Teste**.
- **Arquivos Fontes**. Por exemplo, flags do compilador.
- **Entradas de cache**.
- **Arquivos instalados**.

Para obter uma lista completa de propriedades conhecidas pelo CMake:

.. code-block:: bash

   $ cmake --help-properties | less

Você pode obter o valor atual de qualquer propriedade com:

.. signature:: |get_property|

   .. code-block:: cmake

      get_property(<variable>
             <GLOBAL
              DIRECTORY [<dir>]
              TARGET    <target>
              SOURCE    <source>
                        [DIRECTORY <dir> | TARGET_DIRECTORY <target>]
              INSTALL   <file>
              TEST      <test>
              CACHE     <entry>
              VARIABLE
             PROPERTY <name>
             [SET | DEFINED | BRIEF_DOCS | FULL_DOCS])

e defina o valor de qualquer propriedade com:

.. signature:: |set_property|

   .. code-block:: cmake

      set_property(<GLOBAL
              DIRECTORY [<dir>]
              TARGET    [<target1> ...]
              SOURCE    [<src1> ...]
                        [DIRECTORY <dirs> ...]
                        [TARGET_DIRECTORY <targets> ...]
              INSTALL   [<file1> ...]
              TEST      [<test1> ...]
              CACHE     [<entry1> ...]
             [APPEND] [APPEND_STRING]
             PROPERTY <name> [<value1> ...])


.. _multiple-folders:

Múltiplas pastas
-----------------

Cada pasta em um projeto de várias pastas conterá 
um ``CMakeLists.txt``: uma árvore de origem com 
uma **raiz** e muitas **folhas**.

.. code-block:: text

   project/
   ├── CMakeLists.txt           <--- Root
   ├── external
   │   ├── CMakeLists.txt       <--- Leaf at level 1
   └── src
       ├── CMakeLists.txt       <--- Leaf at level 1
       ├── evolution
       │   ├── CMakeLists.txt   <--- Leaf at level 2
       ├── initial
       │   ├── CMakeLists.txt   <--- Leaf at level 2
       ├── io
       │   ├── CMakeLists.txt   <--- Leaf at level 2
       └── parser
           └── CMakeLists.txt   <--- Leaf at level 2

O script ``CMakeLists.txt`` raiz conterá a invocação do comando |project|: 
variáveis e alvos declarados na raiz têm escopo efetivamente global. 
Lembre-se também que |PROJECT_SOURCE_DIR| apontará para a pasta que contém o ``CMakeLists.txt``raiz.
Para mover-se entre a raiz e uma folha ou entre folhas, 
você usará o comando |add_subdirectory|:

.. signature:: |add_subdirectory|

   .. code-block:: cmake

      add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

Normalmente, você só precisa passar o primeiro argumento: a pasta dentro 
da árvore de compilação será calculada automaticamente pelo CMake.
Podemos declarar alvos em qualquer nível, não necessariamente na raiz: 
um alvo é visível no nível em que é declarado e em todos os níveis superiores.

.. exercise:: Exercício 21: Autômatos celulares

   Vamos além do "Hello, world" e trabalharemos em um projeto que abrange 
   várias pastas. Implementaremos um código relativamente simples para 
   calcular e imprimir na tela `autômatos celulares elementares <https://en.wikipedia.org/wiki/Cellular_automaton#Elementary_cellular_automata>`_.
   Separamos as fontes em ``src`` e ``external`` para simular um projeto 
   aninhado que reutiliza um projeto externo.

   Seu objetivo é:

   - Construa uma biblioteca a partir do conteúdo de ``external`` e de 
     cada subpasta de ``src``. Use |add_library| junto com |target_sources| e, 
     para C++, |target_include_directories|. Pense cuidadosamente sobre 
     os *níveis de visibilidade*.
   - Compile o executável principal. Onde ele está localizado na árvore 
     de construção? Lembre-se de que o CMake gera uma árvore de compilação 
     que espelha a árvore de origem.
   - O executável aceitará 3 argumentos: o comprimento, 
     o número de passos e a regra do autômato. 
     Você pode executá-lo com:

     .. code-block:: bash

        $ automata 40 5 30

     Esta é a saída:

     .. code-block:: text

        length: 40
        number of steps: 5
        rule: 30
                            *
                           ***
                          **  *
                         ** ****
                        **  *   *
                       ** **** ***

   .. tabs::

      .. tab:: C++

         O projeto base está em ``source/code/day-2/21_automata-cxx``.
         As fontes estão organizadas na segunte árvore:

         .. code-block:: text

            automata-cxx/
            ├── external
            │   ├── conversion.cpp
            │   └── conversion.hpp
            └── src
                ├── evolution
                │   ├── evolution.cpp
                │   └── evolution.hpp
                ├── initial
                │   ├── initial.cpp
                │   └── initial.hpp
                ├── io
                │   ├── io.cpp
                │   └── io.hpp
                ├── main.cpp
                └── parser
                    ├── parser.cpp
                    └── parser.hpp

         1. Os arquivos de cabeçalho devem ser incluídos na 
            chamada de |target_sources|? Se sim, qual nível de visibilidade 
            você deve usar?
         2. Em |target_sources|, usar caminhos absolutos (``${CMAKE_CURRENT_LIST_DIR}/parser.cpp``) 
            ou relativos (``parser.cpp``) faz alguma diferença?

         Um exemplo funcional está na subpasta ``solution``.

      .. tab:: Fortran

         O projeto base está em ``content/code/day-2/21_automata-f``.
         As fontes estão organizadas na segunte árvore:

         .. code-block:: text

            automata-f/
            ├── external
            │   └── conversion.f90
            └── src
                ├── evolution
                │   ├── ancestors.f90
                │   ├── empty.f90
                │   └── evolution.f90
                ├── initial
                │   └── initial.f90
                ├── io
                │   └── io.f90
                ├── main.f90
                └── parser
                    └── parser.f90

         1. A fonte ``empty.f90`` declara, como o nome sugere, um módulo 
            Fortran vazio. Este módulo é usado apenas dentro da 
            subpasta ``evolution``: qual nível de visibilidade ele deve 
            ter em |target_sources|?
         2. Observe que o CMake pode entender a ordem de compilação 
            imposta pelos módulos Fortran sem intervenção adicional. 
            Onde estão os arquivos ``.mod``?

         Um exemplo funcional está na subpasta ``solution``.

      .. tab:: Bonus

         Você pode decidir onde executáveis, bibliotecas estáticas 
         e compartilhadas e arquivos Fortran ``.mod`` serão armazenados 
         na árvore de compilação.
         As variáveis relevantes são:

         - ``CMAKE_RUNTIME_OUTPUT_DIRECTORY``, para executáveis.
         - ``CMAKE_ARCHIVE_OUTPUT_DIRECTORY``, para bibliotecas estáticas.
         - ``CMAKE_LIBRARY_OUTPUT_DIRECTORY``, para bibliotecas compartilhadas.
         - ``CMAKE_Fortran_MODULE_DIRECTORY``, Para arquivos Fortran ``.mod``

         Modifique seu ``CMakeLists.txt`` para gerar o executável 
         ``automata`` em `build/bin`` e as bibliotecas em ``build/lib``


.. callout:: A árvore de dependência interna

Você pode visualizar as dependências entre os alvos em seu projeto com o Graphviz:

  .. code-block:: bash

     $ cd build
     $ cmake --graphviz=project.dot ..
     $ dot -T svg project.dot -o project.svg


  .. figure:: img/project.svg
     :align: center

     As dependências entre alvos no projeto de autômatos celulares.


.. keypoints:: Resumo

   - Usando alvos, você pode obter controle granular sobre como os 
     artefatos são criados e como suas dependências são tratadas.
   - *Flags* de compilador, definições, arquivos de origem, pastas de inclusão, 
     bibliotecas de links e opções de vinculador são **propriedades** de um alvo.
   - Evite usar variáveis para expressar dependências entre alvos: 
     use os níveis de visibilidade ``PRIVATE``, ``INTERFACE``, ``PUBLIC`` e 
     deixe o CMake descobrir os detalhes.
   - Use |get_property| para consultar e |set_property| para modificar os valores das propriedades.
   - Para manter a complexidade do sistema de compilação no mínimo, cada pasta em um projeto 
     de várias pastas deve ter seu próprio script CMake.


.. rubric:: Footnotes

.. [#custom_targets]

   Você pode incluir alvos customizados no sistema de construção com |add_custom_target|. 
   Alvos personalizados não são necessariamente artefatos construídos.
