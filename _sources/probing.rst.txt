.. _probing:


A05 - Compilação, vinculação e execução
===========================================

.. questions:: Questão

   - Como você pode adicionar etapas personalizadas ao seu sistema de compilação com CMake?

.. objectives:: Objetivos

   - Aprenda como e quando usar |execute_process|
   - Aprenda a usar |add_custom_command| com alvos.
   - Aprenda a testar a compilação, a vinculação e a execução.


CMake permite que você execute comandos arbitrários em qualquer estágio no ciclo de vida 
do projeto. Este é mais um mecanismo de personalização e discutiremos algumas das 
opções neste tutorial.


Executando comandos personalizados 
-------------------------------------------

O método mais simples é explicitamente executar uma (ou mais) 
processo(s) filhos ao invocar o ``cmake``.  Isso é feito com o
comando |execute_process|.

.. signature:: |execute_process|

   .. code-block:: cmake

      execute_process(COMMAND <cmd1> [args1...]]
                      [COMMAND <cmd2> [args2...] [...]]
                      [WORKING_DIRECTORY <directory>]
                      [TIMEOUT <seconds>]
                      [RESULT_VARIABLE <variable>]
                      [RESULTS_VARIABLE <variable>]
                      [OUTPUT_VARIABLE <variable>]
                      [ERROR_VARIABLE <variable>]
                      [INPUT_FILE <file>]
                      [OUTPUT_FILE <file>]
                      [ERROR_FILE <file>]
                      [OUTPUT_QUIET]
                      [ERROR_QUIET]
                      [OUTPUT_STRIP_TRAILING_WHITESPACE]
                      [ERROR_STRIP_TRAILING_WHITESPACE]
                      [ENCODING <name>])

   Executa um ou mais processos.  A saída padrão e erro padrão
   são registrados nas variáveis ``OUTPUT_VARIABLE`` e ``ERROR_VARIABLE``,
   respectivamente. O resultado do último processo é salvo em
   ``RESULT_VARIABLE``.


É importante notar que qualquer comando invocado pelo ``execute_process``
só será executado na execução do ``cmake``.  


.. exercise:: Exercício 17: Encontre um módulo Python

  Neste exercício, usaremos |execute_process| para verificar se o módulo Python `cffi <https://cffi.readthedocs.io/en/latest/index.html>`_ 
  está instalado em seu ambiente. Na linha de comando, você faria:

   .. code-block:: bash

      $ python -c "import cffi; print(cffi.__version__)"

   Seu objetivo é replicar o mesmo no CMake. O código base está em 
   ``source/code/day-1/17_find_cffi``. 
   Você terá que modificar a chamada de |execute_process| para executar 
   o comando acima. 
   
   Um exemplo funcional está na subpasta ``solution``.

Observe o uso de ``find_package(Python REQUIRED)`` para obter o 
executável ``python``. O CMake vem com muitos módulos dedicados à 
detecção de dependências, como Python. Eles são convencionalmente 
chamados de ``Find<dependency>.cmake`` e você pode inspecionar 
sua documentação com:

.. code-block:: bash

   $ cmake --help-module FindPython | more


Revisitaremos os usos de |find_package| mais tarde em :ref:`dependencies`.


Comandos personalizados para seus alvos (*targets*)
-----------------------------------------------------

Como mencionado, o principal problema de |execute_process| é que vai rodar 
o processo somente quando o comando ``cmake`` é invocado pela primeira vez.
Portanto, *não* é uma alternativa viável se pretendermos realizar algumas 
ações específicas dependendo dos alvos ou tornar o resultado dos comandos 
personalizados uma dependência para outros alvos.

Ambos os casos têm exemplos do mundo real, como ao usar código gerado automaticamente. 
O comando CMake |add_custom_command| pode ser usado em alguns desses casos.

.. signature:: |add_custom_command|

   .. code-block:: cmake

      add_custom_command(TARGET <target>
                   PRE_BUILD | PRE_LINK | POST_BUILD
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [BYPRODUCTS [files...]]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [VERBATIM] [USES_TERMINAL])

   Adicione um ou mais comandos customizados a um destino, como uma biblioteca ou um executável. 
   Os comandos podem ser executados antes da vinculaçao (com ``PRE_BUILD`` e ``PRE_LINK``) 
   ou depois (com ``POST_BUILD``)


.. exercise:: Exercício 18: Antes e depois da montagem

   Queremos realizar alguma ação antes e depois de construir um alvo, neste caso um executável Fortran:

   - Antes de construir, queremos ler a linha de link, conforme produzida pelo CMake, e ecoá-la na saída padrão. 
     Usamos o script Python ``echo-file.py``.
   - Após a construção, queremos verificar o tamanho das alocações estáticas no 
     binário, invocando o comando ``size``. Usamos o script Python ``static-size.py``.

   O código base está em ``sources/code/day-1/18_pre_post-f``.

   #. Adicione comandos do CMake para construir o executável ``example`` 
      das fontes Fortran. Encontre o arquivo de texto com a linha de link 
      na pasta de compilação. Dica: dê uma olhada em ``CMakeFiles`` e tenha 
      em mente o nome que você deu ao alvo.
   #. Chame |add_custom_command| com ``PRE_LINK`` para invocar o script Python ``echo-file.py``.
   #. Chame |add_custom_command| com ``POST_BUILD`` para invocar o script Python ``static-size.py``.

   Um exemplo funcional está na subpasta ``solution``.


Testar a compilação, vinculação e execução
-------------------------------------------

Também queremos poder executar verificações em nossos compiladores e 
vinculadores. Ou verificar se uma determinada biblioteca pode ser usada 
corretamente antes de tentar construir nossos próprios artefatos. 
O CMake fornece módulos e comandos para estes fins:

- ``Check<LANG>CompilerFlag`` fornece a função ``check_<LANG>_compiler_flag``, para 
   verificar se um sinalizador do compilador é válido para o compilador em uso.
- ``Check<LANG>SourceCompiles`` fornece a função ``check_<LANG>_source_compiles`` a qual verifica 
   se um determinado arquivo de origem compila com o compilador em uso.
- ``Check<LANG>SourceRuns`` fornece a função ``check_<LANG>_source_runs`` que certifica se 
    um determinado trecho de origem compila, vincula e executa.

Em todos os casos, ``<LANG>`` pode ser um de ``CXX``, ``C`` ou ``Fortran``.

.. exercise:: Exercício 19: Verifique se um compilador aceita uma flag de compilador

   Os compiladores evoluem: eles adicionam e/ou removem sinalizadores e às 
   vezes você terá que testar se alguns sinalizadores estão disponíveis antes 
   de usá-los em sua compilação.

   O código base está em ``source/code/day-1/19_check_compiler_flag``.

   #. Implemente um ``CMakeLists.txt`` para construir um executável a partir do arquivo fonte ``asan-example.cpp``.
   #. Verifique se os sinalizadores de limpeza de endereço estão disponíveis 
      com |check_cxx_compiler_flag|. As flags a serem verificadas 
      são ``-fsanitize=address -fno-omit-frame-pointer``. 
      Encontre a assinatura do comando com:

      .. code-block:: bash

         $ cmake --help-module CMakeCXXCompilerFlag

   #. Se as flags funcionarem, adicione-os aos usados para compilar o destino 
      executável com |target_compile_options|.

  Um exemplo funcional está na subpasta ``solution``.


.. exercise:: Exercício 20: Testando recursos de tempo de execução

   Testar se alguns recursos funcionarão corretamente para seu código requer não apenas 
   compilar um arquivo de objeto, mas também vincular um executável e executá-lo com êxito.

   O código base está em ``source/code/day-1/20_check_source_runs``.

   #. Crie um destino executável a partir do arquivo fonte ``use-uuid.cpp``.
   #. Adicione uma verificação de que a vinculação à biblioteca produz executáveis 
      funcionais. Use o seguinte código C como teste:

      .. code-block:: c

         #include <uuid/uuid.h>

         int main(int argc, char * argv[]) {
           uuid_t uuid;
           uuid_generate(uuid);
           return 0;
         }

      |check_c_source_runs| requer que o código-fonte de teste seja passado como 
        uma *string*. Encontre a assinatura do comando com:

      .. code-block:: bash

         $ cmake --help-module CheckCSourceRuns

   #.Se o teste for bem-sucedido, vincule o destino executável à biblioteca UUID: use o destino ``PkgConfig::UUID`` como argumento para |target_link_libraries|.

   Um exemplo funcional está na subpasta ``solution``.


.. keypoints:: Resumo

  - Você pode personalizar o sistema de compilação executando comandos personalizados.
  - O CMake oferece comandos para testar a compilação, vinculação e execução.
