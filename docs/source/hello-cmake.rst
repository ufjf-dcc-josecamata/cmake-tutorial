.. _hello-cmake:


A01 - Dos fontes para os executáveis
=====================================

.. questions::

   -Como usamos o CMAKE para compilar arquivos fontes para gerar executáveis?

.. objectives::

   - Aprenda ferramentas disponíveis na suíte Cmake.
   - Aprenda a escrever um simples ``CMakeLists.txt``.
   - Aprenda a diferença entre *build systems*, *build tools*, e *build system generator*.
   - Aprenda a distinguir entre tempos de *configuração*, *geração*, and *construção*.


O que é CMAKE e por que você deveria se importar?
---------------------------------------------------

Sempre que você executar um software, qualquer que seja, desde aplicativos de calendário 
até programas de computação intensivos, haverá um sistema de construção envolvido na 
transformação do código-fonte de texto simples em arquivos binários que podem ser 
executados nos dispositivos que você esteja usando.

Cmake é uma ferramenta open-source que permite gerar automaticamente scripts de construção de aplicação em diferentes plataformas.
Ele fornece uma família de ferramentas e um *linguagem* para **descrever** um o sistema de compilação quando 
as ferramentas de construção apropriadas são invocadas.

A linguagem independe da plataforma *e* do compilador: você pode reutilizar o mesmo 
scripts CMAKE para obter *sistemas de construção nativos* em qualquer plataforma.


.. figure:: img/build-systems.svg
   :align: center

   Em sistemas GNU/Linux,O sistema de construção nativo será uma coleção de ``Makefile``-s.
   A ferramenta de construção ``make`` usa estes ``Makefile``-s para transformar fontes em
   executáveis e bibliotecas.
   CMAke abstrai o processo de gerar o ``Makefile``-s em um script genérico.

Um sistema de compilação baseado em CMAKE:

- Pode trazer seu software mais perto de ser independente de plataforma *e* do compilador.
- tem um bom suporte em muitos ambientes de desenvolvimento integrados (IDES).
- rastreia automaticamente e propaga dependências internas em seu projeto.

Hello, CMake!
-------------

.. typealong:: Compilando "Hello, world" com CMake

   Agora vamos compilar um único arquivo de origem para um executável. Escolha
   sua língua favorita e comece a digitar!

   .. tabs::

      .. tab:: C++

         Você pode encontrar o arquivo com o código-fonte completo na pasta
         ``content/code/day-1/00_hello-cxx``.

         .. literalinclude:: code/day-1/00_hello-cxx/hello.cpp
            :language: c++

         Uma solução está na subpasta ``solution``.

      .. tab:: Fortran

          Você pode encontrar o arquivo com o código-fonte completo na pasta ``content/code/day-1/00_hello-f``.

         .. literalinclude:: code/day-1/00_hello-f/hello.f90
            :language: fortran

         Uma solução está na subpasta ``solution``.

   1. A pasta contém apenas o código-fonte. Precisamos adicionar um arquivo chamado
      ``Cmakelists.txt``. CMAKE lê o conteúdo desses arquivos especiais
      para gerar o sistema de construção.

   2. A primeira coisa que faremos é declarar a exigência de uma versão mínima do CMake:

      .. code-block:: cmake

         cmake_minimum_required(VERSION 3.18)

   3. Em seguida, declaramos nosso projeto e sua linguagem de programação:

      .. code-block:: cmake

         project(Hello LANGUAGES CXX)

   4. Criamos um *alvo executável*. CMAKE gerará regras no sistema de compilação para 
      compilar e vincular nosso arquivo-fonte a um executável:

      .. code-block:: cmake

         add_executable(hello hello.cpp)

   5. Estamos prontos para chamar CMAke e obter o nosso sistema de construção:

      .. code-block:: bash

         $ cmake -S. -Bbuild

   6. E finalmente construimos nosso executável:

      .. code-block:: bash

         $ cmake --build build


Há poucas coisas para notar aqui:

1. Qualquer sistema de compilação CMake invocará os seguintes comandos em sueu ``Cmakelists.txt`` **raiz**
   
   .. signature:: |cmake_minimum_required|

      .. code-block:: cmake

         cmake_minimum_required(VERSION <min>[...<max>] [FATAL_ERROR])

   .. parameters::

      ``VERSION``
          Versão mínima e, opcionalmente, máxima do CMake que deve ser usado.
      ``FATAL_ERROR``
          Levanta um erro fatal se a restrição de versão não estiver satisfeita. Esta 
          ppção é ignorada para versões do CMAKE >= 2.6


   .. signature:: |project|

      .. code-block:: cmake

         project(<PROJECT-NAME>
                 [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
                 [DESCRIPTION <project-description-string>]
                 [HOMEPAGE_URL <url-string>]
                 [LANGUAGES <language-name>...])

   .. parameters::

      ``<PROJECT-NAME>``
         O nome do projeto.
      ``LANGUAGES``
         Linguagem de programação do projeto.

2. Não diferença nos Comandos e variáveis Cmake escritos em minuscúlo ou maiúsculas. 
   No entanto, os arquivos de scripts  **devem ser
   Chamado exatamente** ``CMakeLists.txt``!
3. O comando para adicionar executáveis ao sistema de compilação é, sem surpresa, |add_executable|:

   .. signature:: |add_executable|

      .. code-block:: cmake

         add_executable(<name> [WIN32] [MACOSX_BUNDLE]
                        [EXCLUDE_FROM_ALL]
                        [source1] [source2 ...])

4. Com o CMAKE, você pode abstrair a geração do sistema de compilação e 
   também a invocação das ferramentas de construção.


.. callout:: Coloque o seu ``CMakeLists.txt`` sob controle de versão.

   Todos os arquivos relacionados ao cmake evoluirão junto com sua base de código. 
   Assim, é uma boa ideia colocá-los sob controle de versão. 


.. typealong:: A interface da linha de comando para CMake

   Vamos nos familiarizar com o cmake e especialmente com a sua interface de linha de comando.

   Podemos obter ajuda a qualquer momento com:

   .. code-block:: bash

      $ cmake --help

   Isso produzirá uma série de opções para a tela.
   Podemos analisar as últimas linhas primeiro:

   .. code-block:: text

      Generators

      The following generators are available on this platform (* marks default):
      * Unix Makefiles               = Generates standard UNIX makefiles.
        Green Hills MULTI            = Generates Green Hills MULTI files
                                       (experimental, work-in-progress).
        Ninja                        = Generates build.ninja files.
        Ninja Multi-Config           = Generates build-<Config>.ninja files.
        Watcom WMake                 = Generates Watcom WMake makefiles.
        CodeBlocks - Ninja           = Generates CodeBlocks project files.
        CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
        CodeLite - Ninja             = Generates CodeLite project files.
        CodeLite - Unix Makefiles    = Generates CodeLite project files.
        Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
        Sublime Text 2 - Unix Makefiles
                                     = Generates Sublime Text 2 project files.
        Kate - Ninja                 = Generates Kate project files.
        Kate - Unix Makefiles        = Generates Kate project files.
        Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
        Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.

   Na terminologia Cmake, os scripts nativos de construção e ferramentas de construção 
   são chamados **geradores**. Em uma plataforma específica, a lista mostrará qual 
   ferramentas nativos de construção podem ser usadas através do CMake. 
   Eles podem ser "simples", como ``Makefile``-s ou Ninja, ou projetos IDE.

   A opção  ``-S`` especifica qual diretório de origem o CMake deve processsar,isto é
   a pasta que onde está o ``CMakeLists.txt`` **raiz** do projeto contendo o comando
   |project|.
   Por padrão, o CMake permite  que **os arquivos para a construção** e os artefactos gerados pela compilação
   compilação sejam armazenados ao lado de arquivos-fontes. Isso é **não** uma boa prática: você deve
   sempre manter os artefatos de construção o os fontes separados. Felizmente, o opção ``-B``
   ajuda com isso. Ele deve ser usado para indicar onde armazenar os scripts de construção,
   incluindo o sistema de compilação gerado. Esta é a invocação mínima de ``cmake``:

   .. code-block:: bash

      $ cmake -S. -Bbuild

  Para mudar para outro gerador, usaremos o botão ``-G``:

   .. code-block:: bash

      $ cmake -S. -Bbuild -GNinja

   Opções a serem usadas na geração do sistema de construção são passadas com ``-D``. 
   Por exemplo, para alterar os compiladores:

   .. code-block:: bash

      $ cmake -S. -Bbuild -GNinja -DCMAKE_CXX_COMPILER=clang++

   Finalmente, você pode acessar o manual completo do CMake com:

   .. code-block:: bash

      $ cmake --help-full

   Você também pode perguntar sobre um módulo específico, comando ou variável:

   .. code-block:: bash

      $ cmake --help-variable CMAKE_GENERATOR



Um conjunto de ferramentas completo
--------------------------------------

Cmake oferece uma um ferramental completo para gerenciar o ciclo de desenvolvimento: 
de fontes para scripts/projetos de construão, testes e implantação.
Neste tutorial, vamos discutir:

- **Configuração**: Esse é o estágio quando ``cmake`` é invocado para gerar o 
  sistema de construção a partir de um arquivo ``CMakeLists.txt``.

- **Construção**: Isso é tratado pelas ferramentas de construção nativa

- **Teste**: Nesta fase, você testará os artefatos construidos.


.. figure:: img/cmake-times.jpg
   :align: center

   Você pode gerenciar todos os estágios de uma vida útil de um projeto de software com as ferramentas fornecidas pelo CMAKE.
   Esta figura mostra todas essas etapas e qual ferramenta é apropriada para cada um.
   A figura é reproduzida do `CMake Cookbook
   <https://github.com/dev-cafe/cmake-cookbook>`_ e é licenciado sob o
   termos do `CC-BY-SA
   <https://creativecommons.org/licenses/by-sa/4.0/legalcode>`_.


Produzindo bibliotecas
------------------------

CMake pode ser usado para produzir bibliotecas, bem como executáveis.
O comando que deve ser usado nesse caso é |add_library|:

.. signature:: |add_library|

   .. code-block:: cmake

      add_library(<name> [STATIC | SHARED | MODULE]
                  [EXCLUDE_FROM_ALL]
                  [<source>...])

Você pode vincular bibliotecas em executáveis com |target_link_libraries|:

.. signature:: |target_link_libraries|

   .. code-block:: cmake

      target_link_libraries(<target>
                            <PRIVATE|PUBLIC|INTERFACE> <item>...
                           [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)

.. callout:: Executáveis e bibliotecas são *targets*

   Vamos encontrar o termo **alvo (target)** repetidamente. Em cmake, um alvo é qualquer
   objeto dado como primeiro argumento para |add_executable| ou |add_library|. 
   Sempre que você precisar organizar projetos complexo,  pense em termos de alvos e 
   suas dependências.  
   Toda a família de comandos cmake ``target_*`` Pode ser usado para expressar 
   dependências e é muito mais eficaz do que acompanhar o estado de variáveis.
   Vamos esclarecer esses conceitos em :ref:`targets`.

.. exercise:: Exercício 1: Produzindo bibliotecas

   .. tabs::

      .. tab:: C++

         Você pode encontrar um projeto base na pasta
         ``content/code/day-1/01_libraries-cxx`` .

         #. Escreva um ``CMakeLists.txt`` para compilar os códigos-fontes
            ``Message.hpp`` e  ``Message.cpp`` como uma bibliotecas. 
            Não especifique o tipo de biblioteca, compartilhada ou estática, explicitamente.
         #. Adicione o executável  ``hello-world.cpp``.
         #. Vincule a biblioteca ao executável.

         Uma solução está na subpasta ``solution``.

      .. tab:: Fortran

        Você pode encontrar um projeto base na pasta
         ``content/code/day-1/01_libraries-f`` folder.

         #. Escreva um  ``CMakeLists.txt`` para compilar o código-fonte ``message.f90`` como uma bibliotecas.   
            Não especifique o tipo de biblioteca, compartilhada ou estática, explicitamente.
         #. Adicione o executável ``hello-world.f90`` .
         #. Vincule a biblioteca ao executável.

         Uma solução está na subpasta ``solution``.

  Que tipo de biblioteca você conseguiu?Estático ou compartilhado?

.. keypoints::

   - O CMAKE é um **gerador de sistema de contrução**, e não sistema de compilação.
   - Escrevemos o script ``CMakeLists.txt`` para descrever como as ferramentas de construção criarão artefatos de código-fontes.
   - Você pode usar o conjunto de ferramentas CMAKE para gerenciar toda ciclo de produção de um software vida: de arquivos-fontes até testes e instalação.
   - A estrutura do projeto é espelhada na pasta Build.
