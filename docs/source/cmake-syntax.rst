.. _cmake-syntax:


A02 - Síntaxe CMake
====================

.. questions::

   - Como podemos alcançar mais controle sobre o sistema de construção gerado por cmake?
   - É possível deixar o usuário decidir o que gerar?


.. objectives::

   - Aprenda a definir variáveis com |set| e usá-los com o operador ``${}``
     para `referenciar variáveis
     <https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#variable-references>`_.
   - Aprenda a sintaxe para condicionais em cmake: |if| - ``elseif`` - ``else`` - ``endif``
   - Aprenda a sintaxe para loops em cmake: |foreach|
   - Aprenda como as estruturas CMAKE constroem artefatos.
   - Aprenda a imprimir mensagens úteis.
   - Aprenda a lidar com as opções voltadas para o usuário: |option| e o papel do cache do CMAKE.

CMAKE Oferece um  *idioma específico de domínio* (do ingles, *domain-specific language*, DSL) para descrever como gerar um 
sistema de compilação nativo de uma plataforma específica que você pode deseja executar.
Neste tópico, vamos nos familiarizar com sua sintaxe.


The CMake DSL
--------------

Lembre-se de que o DSL é *insensível a maiúsculas e minúsculas*. 
Agora vamos dar uma olhada nos seus principais elementos.

Variáveis.
++++++++++

Há variáveis definidas pelo cmake ou pelo usuário. Pode-se obter a lista de variáveis definidas pelo 
CMAKE com:

.. code-block:: bash

   $ cmake --help-variable-list

Você pode criar uma nova variável com o comando |set|:

.. signature:: |set|

   .. code-block:: cmake

      set(<variable> <value>... [PARENT_SCOPE])

Variáveis no Cmake são sempre de tipo string, mas certos comandos podem interpretá-los como outros tipos.
Se você quiser declarar uma variável *list*, você terá que fornecer-lhe como uma string separada por 
ponto-e-virgula. Listas podem ser manipuladas com a comando ``list``.

Você pode inspecionar o valor de qualquer variável por *desreferenciação* com o operador ``${}``, 
assim como no *bash*. Por exemplo, o trecho a seguir define o conteúdo
da variável ``hello`` e, em seguida, imprime seu conteúdo:

.. code-block:: cmake

   set(hello "world")
   message("hello ${hello}")

Duas coisas para observar:

- Se a variável dentro do operator ``$ {}``'  não estiver definida, você receberá uma string vazia.
- Você pode referenciar variáveis aninhadas: ``$ {Outer_${inner_variable} _variable}``
  Elas serão avaliadas de dentro para fora.

Um dos aspectos mais confusos no CMAKE é a escopo das variáveis. Existem três escopos de variáveis:

- Função. Escopo quando uma variável é definida (comando |set|) em uma função:
   a variável será visível dentro da função, mas não fora.
- Diretório. Escopo ao processar um ``CMakeLists.txt`` em um diretório:
  Variáveis na pasta pai estarão disponíveis, mas qualquer um que é definida na 
  pasta atual não será propagada para o pai.
- Cache. Essas variáveis são *persistentes* através de chamadas para ``cmake`` e
  disponível para todos os escopos no projeto.
  Modificar uma variável de cache requer o uso de uma forma especial da função |set|:

  .. signature:: |set|

     .. code-block:: cmake

        set(<variable> <value>... CACHE <type> <docstring> [FORCE])


Aqui está uma lista de algumas variáveis definidas pela CMAKE:

- |PROJECT_BINARY_DIR|. Indica a pasta build do projeto.
- |PROJECT_SOURCE_DIR|. Indica a pasta raiz do projeto.
- |CMAKE_CURRENT_LIST_DIR|. Indica a pasta  para o  ``CMakeLists.txt`` que está sendo processado atualmente.


Para obter ajuda sobre uma variável CMAKE faça:

.. code-block:: bash

   $ cmake --help-variable PROJECT_BINARY_DIR


Comandos
++++++++++

São blocos essenciais do CMAKE, pois permitem manipular variáveis.
Eles incluem construções de fluxo de controle e familas de comando ``target_*`` .
Você pode encontrar uma lista completa de comandos disponíveis com:

.. code-block:: bash

   $ cmake --help-command-list

**Funções** e **macros** são construídos no topo dos comandos e
são definidos pelo usuário ou pré-definidos pelo cmake 
Estes são úteis para evitar a scripts CMAke repetitivos.

A diferença entre uma função e uma macro é seu *escopo*:

1. Funções têm seu próprio escopo: variáveis definidas dentro de uma função não são
   propagadas de volta ao chamador.
2. Macros não têm seu próprio escopo: as variáveis do escopo pai podem ser modificadas 
   e novas variáveis no escopo pai podem ser definidas.


Obter ajuda em um comando específico, função ou macro pode ser obtido com:

.. code-block:: bash

   $ cmake --help-command target_link_libraries



Módulos
+++++++++

São coleções de funções e macros definidos pelo usuário ou pelo cmake. 
Cmake vem com um rico ecossistema de módulos e você provavelmente escreverá alguns
para encapular funções ou macros freqüentemente usados em seus scripts cmake.

Para usar um módulo, voce deverá usar o comando |include|. Veja o exemplo abaixo:

.. code-block:: cmake

   include(CMakePrintHelpers)


A lista completa de módulos internos está disponível com:

.. code-block:: bash

   $ cmake --help-module-list


Ajuda em um módulo integrado específico pode ser obtido com:

.. code-block:: bash

   $ cmake --help-module CMakePrintHelpers


A árvore de construção
------------------------

É intuitivo navegar na pasta Build do projeto: 


.. code-block:: bash

   $ tree -L 2 build

   build
   ├── CMakeCache.txt
   ├── CMakeFiles
   │   ├── 3.18.4
   │   ├── cmake.check_cache
   │   ├── CMakeDirectoryInformation.cmake
   │   ├── CMakeOutput.log
   │   ├── CMakeTmp
   │   ├── compute-areas.dir
   │   ├── geometry.dir
   │   ├── Makefile2
   │   ├── Makefile.cmake
   │   ├── progress.marks
   │   └── TargetDirectories.txt
   ├── cmake_install.cmake
   ├── compute-areas
   ├── libgeometry.a
   └── Makefile

Note que:

- O projeto foi configurado com o gerador ``makefile``.
- A cache é um arquivo texto ``CMakeCache.txt``.
- Para cada alvo (*target*)no projeto, CMake criará uma subpasta
  ``<target>.dir`` em ``CMakeFiles``. Os arquivos de objeto intermediários são
  armazenado nessas pastas, juntamente com flags do compilador e linha de links.
- Os artefatos de construção, ``compute-areas`` e ``libgeometry.a``,  são armazenados na
  raiz da árvore de  construção (*build*).

Controle de fluxo
-------------------

Os comandos |if| e |foreach|  estão disponíveis como construções de controle de fluxo no
Cmake e você certamente está familiarizado com seu uso em outras linguagens de programação

Desde que *todas* as variáveis no cmake são strings, a sintaxe para |if| e | foreach |
aparecem em algumas variantes.

.. signature:: |if|

   .. code-block:: cmake

      if(<condition>)
        # <commands>
      elseif(<condition>) # optional block, can be repeated
        # <commands>
      else()              # optional block
        # <commands>
      endif()

O valor da verdade das condições no blocos |if| e ``elseif`` é
determinado por operadores booleanos. No CMAKE:

- Verdadeiro é qualquer expressão avaliado para: ``1``, ``ON``, ``TRUE``, ``YES``, e
  ``Y``.
- Falso é qualquer expressão avaliado para: ``0``, ``OFF``, ``FALSE``, ``NO``,
  ``N``, ``IGNORE``, e ``NOTFOUND``.

Cmake oferece operador booleano para comparações de string, tal como ``STREQUAL`` por
igualdade de strings, e para comparações de versão,  como  ``VERSION_EQUAL``.

.. callout:: Expansões de variáveis em condicionais

   O comando |if| expande o conteúdo das variáveis antes de avaliar seu valor de verdade.
   Veja a `documentação oficial <https://cmake.org/cmake/help/latest/command/if.html?highlight=#variable-expansion>`_
   para mais detalhes.


.. exercise:: Exercício 2: Condicionais no Cmake

   Modique o ``CMakeLists.txt``do exercicio anterior para contruir uma biblioteca
   *statica* ou *compartilhada*  dependendo do valor do booleano
   ``MAKE_SHARED_LIBRARY``:

   1. Define a variável ``MAKE_SHARED_LIBRARY``.
   2. Escreva uma verificação condicional da variável. Dependendo da condição, chame adequadamente a funçao |add_library| .

   .. tabs::

      .. tab:: C++

         Você pode encontrar um projeto de base em ``content/code/day-1/02_conditionals-cxx``.
         Uma solução está na subpasta ``solution``.

      .. tab:: Fortran

        Você pode encontrar um projeto de base em ``content/code/day-1/02_conditionals-f``.
          Uma solução está na subpasta ``solution``.


Você pode executar a mesma operação em uma coleção de itens com |foreach|:

.. signature:: |foreach|

   .. code-block:: cmake

      foreach(<loop_var> <items>)
        # <commands>
      endforeach()

A lista de itens pode ser sepradados por espaço ou ponto-e-virgula. ``break()`` e
``continue()`` também estão disponíveis.

.. typealong:: Loops no CMake

   Aqui, vamos mostrar como usar |foreach| e listas no CMake. Vamos trabalhar a partir de um projeto base  ``content/code/day-1/03_loops-cxx``.
   O objetivo é compilar uma biblioteca com varios arquivos: alguns deles
   devem ser compilados com nível de otimização ``-O3``, enquanto alguns outros com ``-O2``.
  
   Vamos definir as *flags* de compilação como propriedades no destino da biblioteca.
   Alvos e propriedades serão discutidos em maior profundidade em :ref:`targets`.

   Uma solução está na subpasta ``solution``.


Mensagens de impressão
------------------------

Você provavelmente terá que depurar seus scripts CMAKE em algum momento.
Acreditamos que a depuração de impressão é a maneira mais eficaz de
fazê-lo e para isso podemos usar o coamnndo |message|:

.. signature:: |message|

   .. code-block:: cmake

      message([<mode>] "message to display")

.. parameters::

   ``<mode>``
       Que tipo de mensagem é exibida, por exemplo:

         - ``STATUS``, para informações incidentais.
         - ``FATAL_ERROR``, para relatar um erro que impede a continuidade do processamento.


O comando |message| pode ser um pouco estranho para se trabalhar, especialmente quando você deseja imprimir o nome *e* valor de uma variável.  
Incluindo o módulo embutido ``CMakePrintHelpers`` vai facilitar sua vida ao depurar, já que
fornece o função |cmake_print_variables|:

.. signature:: |cmake_print_variables|

   .. code-block:: cmake

      cmake_print_variables(var1 var2 ... varN)

Este comando aceita um número arbitrário de variáveis e imprime seu nome e valor para a saída padrão.
   Por exemplo:

   .. code-block:: cmake

      include(CMakePrintHelpers)

      cmake_print_variables(CMAKE_C_COMPILER CMAKE_MAJOR_VERSION DOES_NOT_EXIST)

dá:

   .. code-block:: text

      -- CMAKE_C_COMPILER="/usr/bin/gcc" ; CMAKE_MAJOR_VERSION="2" ; DOES_NOT_EXIST=""


Controlando a construção com opções
-------------------------------------

Nós mencionamos anteriormente que o ``-D`` empregado ma interface da linha de comando (CLI)
do comando ``cmake``pode ser usado para passar opções. Entretanto, como definimos estas
opções no nosso ``cmakelists.txt``.

É aí que o comando |option| entra em jogo!

.. signature:: |option|

   .. code-block:: cmake

      option(<variable> "<help_text>" [value])

  Com isso, você pode fornecer uma alternância ON/OFF controlável pelo CLI.

Importando o modulo ``CMakeDependentOption``,você pode lidar com casos onde
as opções são relevantes apenas *se* outras opções já estão definidas 
para valores específicos:

.. signature:: |cmake_dependent_option|

   .. code-block:: cmake

      cmake_dependent_option(USE_FOO "Use Foo" ON
                             "USE_BAR;NOT USE_ZOT" OFF)

  Se a opção ``USE_BAR`` é verdade e a opção ``USE_ZOT`` é falso, então
  a opção ``USE_FOO`` será apresentado ao usuário e será verdadeiro por
  predefinição. Se a condição em ``USE_BAR`` e ``USE_ZOT`` não é realizado, o
  opção é definida como false.


.. exercise:: Exercício 4: Opções voltadas para o usuário

   Neste exercício, vamos trabalhar com |option| e |cmake_dependent_option|.
   Queremos permitir que o usuário decida se deve construir uma biblioteca e se
   deve ser estática ou compartilhada.

   1. Adicione a opção ``USE_LIBRARY``.
   2. Adicione opções dependentes ``MAKE_STATIC_LIBRARY`` e ``MAKE_SHARED_LIBRARY``.
      Eles só serão apresentados se ``USE_LIBRARY`` for verdadeiro.
   3. Use condicionais para orquestrar a construção da biblioteca estática/compartilhada.

   .. tabs::

      .. tab:: C++

         Você pode encontrar um projeto base na pasta ``content/code/day-1/04_options-cxx``.
         Uma solução está na subpasta ``solution``.

      .. tab:: Fortran

         Você pode encontrar um projeto base na pasta ``content/code/day-1/04_options-f``.
         Uma solução está na subpasta  ``solution``.


.. keypoints::

   - Cmake oferece um DSL completo que capacita  a escrita de  ``CMakeLists.txt`` complexos.
   - Variáveis têm regras de escopo.
   - A estrutura do projeto é espelhada na pasta Build.
