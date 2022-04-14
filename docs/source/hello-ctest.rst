.. _hello-ctest:


A03 - Criando e executando testes com o CTest
==============================================

.. questions::

   - Como podemos lidar com o estágio de testes do nosso projeto com CMake?

.. objectives::

   - Aprenda a produzir executáveis de teste com cmake.
   - Aprenda a executar seus testes através do CTest.


O teste é uma atividade essencial no ciclo de desenvolvimento.Um conjunto de testes bem 
projetado irá ajudá-lo a detectar *bugs* e também pode facilitar a integração de novos
desenvolvedores.

.. figure:: img/ctest.svg
   :align: center

   O CTest faz parte do conjunto de programas CMake. É um *executor de testes*. Você pode
   definir um conjunto de testes, executá-los e  gerar relatórios através dele.

Nesta atividade, vamos analisar como usar o CTest para definir e executar nossos testes.

Adicionando testes ao seu projeto
-----------------------------------

No CMake e CTest, um teste é qualquer comando retornando um código de saída. 
Não importa como o comando é emitido ou executado: pode ser um executável C++ ou um script Python.
Contanto que a execução retorne um código de saída zero ou não-zero, o CMake poderá classificar 
se o teste foi bem sucedido ou se falhou, respectivamente.

Existem duas etapas para integrar seu sistema de compilação CMake com a ferramenta CTest:

1. Chamar o comando  ``enable_testing`` a qual não requer argumentos.
2. Adicionar testes com o comando |add_test|.

.. signature:: |add_test|

   .. code-block:: cmake

      add_test(NAME <name> COMMAND <command> [<arg>...]
         [CONFIGURATIONS <config>...]
         [WORKING_DIRECTORY <dir>]
         [COMMAND_EXPAND_LISTS])

   Este comando aceita apenas os *argumentos nomeados*, ``NAME`` e ``COMMAND``
   obrigatoriamente. O primeiro especifica o nome de identificação do teste, enquanto o
   o último configura o comando de execução.


.. typealong:: Seu primeiro projeto de teste

   Vamos construir uma simples biblioteca para somar inteiros e um executável usando esta
   biblioteca.  Use o projeto base localizado na pasta
   ``content/code/day-1/05_hello-ctest``.

   .. code-block:: cmake

      cmake_minimum_required(VERSION 3.18)

      project(hello-ctest LANGUAGES CXX)

      set(CMAKE_CXX_STANDARD 14)
      set(CMAKE_CXX_EXTENSIONS OFF)
      set(CMAKE_CXX_STANDARD_REQUIRED ON)

      add_library(sum_integers sum_integers.cpp)

      add_executable(sum_up main.cpp)
      target_link_libraries(sum_up PRIVATE sum_integers)

   Juntamente com a biblioteca e o executável principal, também produziremos um
   executável para testar a biblioteca ``sum_integers``.

   .. code-block:: cmake

      add_executable(cpp_test test.cpp)
      target_link_libraries(cpp_test PRIVATE sum_integers)

   Agora é hora de configurar o CTest:

   .. code-block:: cmake

      enable_testing()

   e declarar nosso teste, especificando qual comando para executar:

   .. code-block:: cmake

      add_test(
        NAME cpp_test
        COMMAND $<TARGET_FILE:cpp_test>
      )

   Observe o uso de `gerador de expressão (gen-exp)
   <https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html>`_
   para evitar especificar o caminho completo para o executável ``cpp_test``.

   Podemos agora compilar e executar nosso teste:

   .. code-block:: bash

      $ cmake -S. -Bbuild
      $ cd build
      $ cmake --build .
      $ ctest

  Uma solução está na subpasta ``solução``.

.. exercise:: Exercício 6: Executando os testes através de um script shell

   Qualquer comando pode ser usado para executar testes. 
   Neste exercício, vamos estender o código cmake anterior para testar o executável principal 
   dentro de um script de shell.
   Use o projeto base localizado na pasta ``content/code/day-1/06_bash-ctest``.

   #. Encontre o executável bash apropriado para executar ``test.sh``. 
      Você deve usar o comando ``find_program`` do CMake.

      .. code-block:: cmake

         find_program(BASH_EXECUTABLE NAMES bash REQUIRED)

      A variável ``BASH_EXECUTABLE`` será o programa shell.

   #. Adicione outra invocação de |add_test| que será equivalente a executar:

      .. code-block:: bash

         $ ./test.sh sum_up

      Sugestões:

      - Use a localização absoluta de ``test.sh``.
      - Você pode usar a sintaxe gerador de expressão para dar a localização do
        executável: ``$<TARGET_FILE:sum_up>``.

   #. Construa o projeto e execute o CTest.

  Uma solução está na subpasta ``solution``.

.. exercise:: Exercício 7: Executando os testes através de um script Python

   É muito mais comum hoje em dia usar python, em vez de scripts de shell.  
   Neste exercício, adicionaremos mais dois testes ao nosso projeto. 
   Esses novos testes executarão o programa principal por meio de um script Python.
   Use o projeto base localizado na pasta ``content/code/day-1/07_python-ctest``.

   #. Encontre o intérprete Python para executar ``test.py``. Você deve usar 
   o comando |find_package| do CMake:

      .. code-block:: cmake

         find_package(Python REQUIRED)

      O intérprete estará disponível na variável ``Python_EXECUTABLE``.

   #. Adicione outra invocação de |add_test| que será equivalente a executar:

      .. code-block:: bash

         $ python test.py --executable sum_up

      Sugestões:

      - Use a localização absoluta de ``test.py``.
      - Você pode usar a sintaxe gerador de expressão para dar a localização do
        executável:``$<TARGET_FILE:sum_up>``.

   #. O script ``test.py``  aceita o comando por linha de argumento  ``--short``. 
      Adicione outro teste que use essa opção no comando.
   #. Construa o projeto e corra o CTest.

   Uma solução está na subpasta ``solution``.

A interface de linha de comando do CTest
-------------------------------------------

.. typealong:: Como usar o CTest efetivamente.

   Agora, demonstraremos a interface de linha de comando da CTest (CLI) 
   usando a solução do exercício anterior.

   O comando ``ctest`` faz parte da instalação do CMake. Podemos encontrar ajuda em seu uso com:

   .. code-block:: bash

      $ ctest --help

   **Lembre-se**, para executar seus testes através do CTEST, primeiro 
   você precisará estar na pasta de compilação:

   .. code-block:: bash

      $ cd build
      $ ctest

   Isso executará todos os testes em seu suíte de teste.
   Você pode listar os nomes dos testes no conjunto de testes com:

   .. code-block:: bash

      $ ctest -N

   As opções de verbosidade também são muito úteis, especialmente quando a depuração de falhas.
   Com ``--output-on-failure``, o CTest imprimirá a saída de testes com falha.
   Se você gostaria de exibir a invocação completa para cada teste, use
   tha opção ``--verbose``.
   Pode-se selecionar *subconjuntos* de testes para executar:

   - Por *nome*, com a flag ``-R <regex>``. Qualquer teste cujo *nome* pode ser
     capturado pelo *regex* será executado.  A opção ``-RE <regex>`` 
     *exclui* testes por nome usando um regex.
   - Por *label*, com a flag ``-R <regex>``. Qualquer teste cujo *label* pode ser
     capturado pelo *regex* será executado.  A opção ``-RE <regex>`` 
     *exclui* testes por label usando um regex.
   - Por *numero*, com a flag ``-I [Start,End,Stride,test#,test#|Test file]``.
     Esta geralmente não é a opção mais conveniente para selecionar subconjuntos de testes.

   É possível executar testes com falha com:

   .. code-block:: bash

      $ ctest --rerun-failed

   Finalmente, você pode paralelizar a execução do teste:

   .. code-block:: bash

      $ ctest -j N
      $ ctest --parallel N

   **Cuidado!** A ordem de execução de testes não é garantida: se alguns testes são interdependentes, você terá que declarar explicitamente em sua construção.


Propriedades do teste: rótulos, tempo limite e custo
-------------------------------------------------------

Quando você usa |add_test|, Você dá um nome único para cada teste. Como nós vimos,
Você pode usar esses nomes para filtrar quais testes serão executados na suíte. 
Isso pode ser extremamente valioso quando a suíte de teste é muito grande e você  precisará verificar
apenas um subconjuntos dos testes.
No entanto, o mecanismo de nomenclatura não permite que os testes sejam agrupados facilmente. 
Poderíamos, em princípio, adicionar um sufixo a todos os testes em um determinado grupo e depois filtrá-los com
um regex apropriado, mas e se tivéssemos vários grupos para os quais os testes pudessem
pertencer. Esta é uma situação muito comum na prática!
Felizmente, podemos definir **propriedades** noson testes e os rótulos (*labels*) estão entre os
propriedades disponíveis.

.. signature:: |set_tests_properties|

   .. code-block:: cmake

      set_tests_properties(test1 [test2...] PROPERTIES prop1 value1 prop2 value2)


.. exercise:: Exercício 8: Definir etiquetas em testes


   Vamos executar alguns testes usando Python e queremos agrupá-los em duas categorias:

   - ``quick`` para testes com um tempo de execução muito curto.
   - ``long`` para testes de benchmarking com um tempo de execução mais longo.

   Use o projeto base localizado na pasta ``content/code/day-1/08_ctest-labels``.

   .. tabs::

      .. tab:: Labeling

         1. Encontre o interpretador Python:

            .. code-block:: cmake

               find_package(Python REQUIRED)

            O interpretador estará disponível na variável ``Python_EXECUTABLE``.
         2. Enable testing.
         3. Adicione os seis testes da pasta ``test``. Dê a cada um deles um nome único.
         4. Use |set_tests_properties| para definir rótulos para os testes:

            - ``feature-a.py``, ``feature-b.py``, e ``feature-c.py`` devem ser do grupo ``quick``.
            - ``feature-d.py``, ``benchmark-a.py``, e ``benchmark-b.py``
              devem ser do grupo ``long``.

         5. Verifique se tudo funciona como esperado

      .. tab:: Bonus

         Tente simplificar as chamadas repetidas para |add_test| com o comando |foreach|.
         Você pode precisar aplicar algumas manipulações no nome dos arquivos: confira o
         comando |file|.

   Uma solução está na subpasta ``solution``.

Entre as muitas propriedades que podem ser definidas em testes, gostaríamos de destacar as seguintes:

- ``WILL_FAIL``. O CTest marcará os testes como ``OK`` quando o comando correspondente
  retorna com um código de saída diferente de zero. Use esta propriedade para testar falhas esperadas.
- ``COST``. Na primeira vez que você executa seus testes, o CTest arazenará o tempo de execução de cada um. 
   Desta forma, as execuções subseqüentes do conjunto de testes começarão a partir dos testes com maiores tempo de execução. 
- ``TIMEOUT``. Alguns testes podem ser executados por um longo período: você pode definir um tempo limite explícito se quiser ser mais ou menos tolerante das variações de tempo na execução


.. exercise:: Exercícios 9, 10, 11: Mais propriedades!

  Vamos brincar com as propriedades que acabamos de introduzir.

   .. tabs::

      .. tab:: WILL_FAIL

         Use o projeto base localizado na pasta
         ``content/code/day-1/09_ctest-will-fail``.

         1. Crie um projeto sem uma linguagem.
         2. Encontre o interpretador Python.
         3. Habilite testing.
         4. Adicione script de teste ``test.py``.

         Tente executar os testes e observe o que acontece.  
         Agora defina a propriedade ``WILL_FAIL`` para verdade e observe o 
         que muda ao executar os testes.

         Uma solução está na subpasta ``solution``.

      .. tab:: COST

         Use o projeto base localizado na pasta
         ``content/code/day-1/10_ctest-cost``.

         1. Ativar testes no ``CMakeLists.txt``.
         2. Adicionar testes em execução a cada um dos scripts na pasta ``test``.
         3. Execute os testes em paralelo e observe quanto tempo sua execução leva.
         4. Re-execute os testes e observe como o CTest ordena sua execução.
         5. Agora defina a propriedade ``COST``. O que mudou quando re-executou os testes.

         Uma solução está na subpasta ``solution``.

      .. tab:: TIMEOUT

         Use o projeto base localizado na pasta
         ``content/code/day-1/11_ctest-timeout``.

         1. Crie um projeto sem uma linguagem.
         2. Encontre o interpretador Python.
         3. Habilite testing.
         4. Adicione script de teste ``test.py``.

         Tente executar os testes e observar quanto tempo o teste leva para executar.
         Agora defina o. ``TIMEOUT`` para um valor *menor* do que você observou e re-execute os testes.

         Uma solução está na subpasta ``solution``.

Para obter uma lista completa de propriedades que podem ser definidas faça:

.. code-block:: bash

   $ cmake --help-properties

ou visite a documentação cmake `online <https://cmake.org/cmake/help/v3.19/manual/cmake-properties.7.html#properties-on-tests>`_.



.. keypoints::

   - Qualquer comando pode ser definido como um teste no cmake.
   - Os testes podem ser executados através do CTest.
