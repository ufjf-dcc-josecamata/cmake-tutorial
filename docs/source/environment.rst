.. _environment:


A04 - Detectando seu ambiente
==================================

.. questions:: Questão

   - Como o CMake interage com o ambiente?

.. objectives:: Objetivos

   - Aprenda como descobrir o sistema operacional.
   - Aprenda a descobrir características do processador.
   - Aprenda a lidar com o código-fonte dependente de plataforma e compilador.


O CMake vem pré-configurado com padrões  para uma 
infinidade de propriedades dos ambientes em que pode ser usado. 
Gerador padrão, compilador padrão e *flags* de compilador 
são os mais notáveis exemplos dessa configuração inicial.
Execute o seguinte:

.. code-block:: bash

   $ cmake --system-information | less


Se você estiver curioso sobre que tipo de configuração é fornecida para 
sua versão do CMake e sistema operacional, nesta atividade, mostraremos 
como usar o CMake para fazer uma introspecção no ambiente em que 
estamos rodando. Isso é muito comum em sistemas de construção, 
pois permite customizar a criação de artefatos em tempo real.


.. typealong:: Descobrindo o sistema operacional

   Para este exemplo, usamos o valor especial ``NONE`` para a opção ``LANGUAGES``: 
   estamos interessados apenas em relatar qual sistema operacional o 
   CMake descobre e isso é independente da linguagem de programação.

   Você pode encontrar o exemplo completo e funcional em
   ``source/code/day-1/12_OS/solution``.


Descobrindo o processador
---------------------------

Uma personalização comum é aplicar *flags* de compilador 
específicos do processador. Podemos obter essas informações no 
sistema host com o comando built-in |cmake_host_system_information|.

.. signature:: |cmake_host_system_information|

   .. code-block:: cmake

      cmake_host_system_information(RESULT variable QUERY <key> ...)

   Este comando aceita uma ou mais consultas no sistema *host* e retorna o resultado em ``variable``.


.. typealong:: Descoberta do processador

   Mais uma vez, usamos o valor especial ``NONE`` para a opção ``LANGUAGES``:
   Estamos interessados apenas em relatar o que o CMake descobre 
   sobre o sistema host e isso é independente da linguagem de programação.

   Você pode encontrar o exemplo completo e funcional em
   ``source/code/day-1/13_processor/solution``.


.. exercise:: Exercício 14: Conheça o seu host

   O projeto é baseado no modelo anterior e pode ser encontrado na pasta
   ``source/code/day-1/14_host_system_information``.

   #. Abra a página de ajuda para |cmake_host_system_information|. 
      Tanto no navegador quanto na linha de comando:

      .. code-block:: bash

         $ cmake --help-command cmake_host_system_information

   #. Estenda o código do base para consultar todas as chaves listadas na página de ajuda e imprima-as.

  Uma solução funcional está na subpasta ``solution``.


Código-fonte dependente da plataforma e do compilador
-------------------------------------------------------


.. typealong:: Compilação condicional com definições de pré-processador

  Às vezes, precisamos escrever um código que execute diferentes operações 
  com base em constantes de tempo de compilação:

   .. code-block:: c++

      #ifdef IS_WINDOWS
        return std::string("Hello from Windows!");
      #elif IS_LINUX
        return std::string("Hello from Linux!");
      #elif IS_MACOS
        return std::string("Hello from macOS!");
      #else
        return std::string("Hello from an unknown system!");
      #endif

   Podemos conseguir isso com o CMake com uma combinação de introspecção 
   do sistema host com o comando |target_compile_definitions|.

   Uma solução funcional está na subpasta ``source/code/day-1/15_sys_preproc/solution``.

.. signature:: |target_compile_definitions|

   .. code-block:: cmake

      target_compile_definitions(<target>
        <INTERFACE|PUBLIC|PRIVATE> [items1...]
        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

   Adiciona uma (ou mais) definições de compilação ao ``<target>``.

Pode ser mais conveniente ter um único arquivo contendo todas essas
constantes de tempo de compilação, em vez de passá-las para o 
pré-processador. Isso pode ser feito tendo um arquivo *configuravel* 
pelo CMake depois de descobrir os valores para todas as constantes 
de tempo de compilação necessárias.

.. signature:: |configure_file|

   .. code-block:: cmake

      configure_file(<input> <output>
                     [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
                     [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])

   Copia o arquivo ``<input>`` para outro arquivo ``<output>``, modificando seu conteúdo.

.. parameters::

   ``<input>``
      Caminho para o arquivo de entrada contendo espaços reservados 
      (``@placeholder@``) a ser substituído pelo conteúdo da variável 
      CMake correspondente. Um caminho relativo é tratado em relação 
      ao valor de ``CMAKE_CURRENT_SOURCE_DIR``.
   ``<output>``
      Caminho para o arquivo ou diretório de saída. Um caminho relativo 
      é tratado em relação ao valor de ``CMAKE_CURRENT_BINARY_DIR``.


.. exercise:: Exercício 16: Configurar um arquivo

   Vamos revisitar um dos exercícios anteriores. Em vez de imprimir os 
   resultados da consulta com |cmake_host_system_information|, queremos 
   salvar os resultados em um arquivo de cabeçalho e usá-lo para 
   imprimir os resultados ao executar um programa.


   O código fonte para este projeto está na pasta 
   ``content/code/day-1/16_configure``. O arquivo de cabeçalho 
   ``config.h.in`` contém espaços reservados para os valores que 
   detectaremos usando o CMake:  número de núcleos físicos, 
   memória física total, nome do SO e plataforma do SO.

   #. Copie-cole o ``CMakeLists.txt`` da pasta 
      ``content/code/day-1/14_host_system_information``.
   #. Adapte o ``CMakeLists.txt`` para compilar ``processor-info.cpp`` em um
      executável comm |add_executable|.
   #. Experimente construir. Isso deve falhar, porque ainda não há arquivo ``config.h`` 
      em nenhum lugar!
   #. Abra a página de ajuda para |configure_file|. No navegador ou na linha de comando:

      .. code-block:: bash

         $ cmake --help-command configure_file

   #. Consulte as chaves (número de núcleos físicos, memória física total, nome do SO e plataforma do SO) 
      usando os nomes correspondentes listados na página de ajuda para
      |cmake_host_system_information| e salve-os em variáveis nomeadas 
      de forma apropriada.
   #. Invoque |configure_file| para produzir ``config.h`` de ``config.h.in``.
      O arquivo configurado será salvo na pasta build, *ou seja*, em ``PROJECT_BINARY_DIR``.
   #. Tente construir novamente. Isso também falhará, porque o cabeçalho não 
      está no *caminho de inclusão*. Podemos corrigir isso com:

      .. code-block:: cmake

         target_include_directories(processor-info
           PRIVATE
             ${PROJECT_BINARY_DIR}
           )

   Uma solução funcional está na subpasta ``solution``.


.. keypoints:: Resumo

   - O CMake pode *introspectar* o sistema host.
   - Você pode compilar o código-fonte de forma diferente, com base no sistema operacional, no processador,  no compilador ou em qualquer combinação deles.
   - Você pode gerar o código-fonte ao configurar o projeto com |configure_file|.
