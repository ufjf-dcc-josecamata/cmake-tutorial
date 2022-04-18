.. _python-bindings:

A09 - Misturando Python e linguagens compiladas
================================================

.. questions:: Questão

   - Como podemos lidar com projetos multilinguagens com CMake?

.. objectives:: Objetivos

   - Aprenda a construir ligações Python pybind11.
   - Aprenda a construir ligações CFFI Python.

Python é uma linguagem de programação dinâmica flexível. 
Como o próprio Python é escrito na linguagem de programação C, é possível 
escrever módulos de *extensão* em uma linguagem compilada. 
Obtém-se toda a flexibilidade evitando as penalidades de desempenho 
inerentes às linguagens interpretadas. Muitos frameworks estão disponíveis 
para preencher a lacuna entre linguagens compiladas e
Python. Todos eles contam com alguma forma de geração automática de código:

- `SWIG <http://swig.org/>`_. Possivelmente o framework com a história mais longa.
- `CFFI <https://cffi.readthedocs.io/en/latest/index.html>`_. Funciona com C e Fortran.
- `Cython <https://cython.org/>`_. Funciona com C e pode exigir muito esforço.

Misturando C++ e Python com pybind11
++++++++++++++++++++++++++++++++++++++

Se você está escrevendo C++, você tem ainda mais opções de frameworks de vinculação:

- `Boost.Python
  <https://www.boost.org/doc/libs/1_75_0/libs/python/doc/html/index.html>`_.
  Adaptado para C++ e conta com metaprogramação de modelo para gerar ligações em tempo de compilação.

- `pybind11 <https://pybind11.readthedocs.io/en/stable/index.html>`_. Mesma filosofia do Boost.Python, mas projetado para C++11 e além.

Se você escrever *modern C++*, pybind11 deve ser sua estrutura de escolha:

- É uma biblioteca somente de cabeçalho e, portanto, uma dependência bastante fácil de satisfazer.
- O código de ligação será bastante compacto: você não terá que manter uma base de código excessivamente grande.
- Possui excelente integração com o CMake.


.. exercise:: Exercício 27: Código bancário com C++ e Python

   Nosso objetivo é compilar wrappers Python para uma pequena 
   biblioteca C++ simulando uma conta bancária. A dependência pybind11 
   será satisfeita no momento da configuração usando ``FetchContent``.

   O projeto base está em ``source/code/day-2/27_cxx-pybind11``.
   A estrutura da pasta é a seguinte:

   .. code-block:: text

      27_cxx-pybind11
      └── account
          ├── account.cpp
          ├── account.hpp
          └── test.py

   #. Crie um ``CMakeLists.txt`` na raiz do programa, com 
      requisito e projeto mínimo do CMake.
   #. Encontre o Python com |find_package|. Solicite pelo menos a versão 
      3.6 com a palavra-chave ``REQUIRED`` e o interpretador e 
      os cabeçalhos de desenvolvimento com a palavra-chave 
      ``COMPONENTS``. Consulte a documentação:

      .. code-block:: bash

         $ cmake --help-module FindPython | less

   #. Habilite o teste e adicione a pasta ``account``.
   #. Preencha o ``CMakeLists.txt`` base na pasta ``account``, 
      seguindo os prompts do ``FIXME``. Queremos baixar o tarball 
      lançado para a versão 2.6.2 do pybind11.
   #. Configure, construa e execute o teste.

  Uma solução funcional está na subpasta ``solution``.

   .. note::

      - A função ``pybind11_add_module`` é um wrapper de conveniência para |add_library| para 
        gerar módulos de extensão Python. É oferecido por pybind11 e você 
        pode ler mais sobre isso `aqui
        <https://pybind11.readthedocs.io/en/stable/compiling.html#pybind11-add-module>`_.
      - A sintaxe especial usada na definição do comando test definirá o local da 
        extensão Python como uma variável de ambiente.


Misturando C/Fortran e Python com CFFI
+++++++++++++++++++++++++++++++++++++++

`CFFI <https://cffi.readthedocs.io/en/latest/index.html>`_, abreviação de "C Foreign Function Interface", é um módulo Python que 
ajuda na criação de interfaces Python para projetos C-interoperáveis.
Usar CFFI pode ser um pouco mais simples do que trabalhar com pybind11. 
No entanto, ele permite que você crie interfaces Python para projetos 
Fortran de forma mais direta do que com Cython ou SWIG.
Isso requer alguns passos:

#. escrever um arquivo de cabeçalho C definindo a 
   interface de programação de aplicativos (API) do seu código.
#. invocando CFFI para analisar o arquivo de cabeçalho da API e 
   produzir o código de wrapper C correspondente.
#. compilando o código wrapper gerado em um módulo Python.

Embora a etapa 1 dependa do código para o qual você deseja fornecer o 
código de vinculações do Python, as etapas 2 e 3 podem ser automatizadas 
em um sistema de compilação CMake.

.. exercise:: Exercício 28: Código bancário usando CFFI

   Nosso objetivo é compilar wrappers Python para uma pequena biblioteca simulando uma conta bancária. 
   O código de exemplo já tem um arquivo de cabeçalho da API.
   Este exercício mostrará como realizar as etapas 2 e 3 acima:

   - O script Python ``cffi_builder.py`` analisa o arquivo de cabeçalho da 
      API e irá gerar o arquivo fonte ``_pyaccount.c`` no 
      *tempo de compilação*. Conseguimos isso no CMake usando um 
      comando personalizado, emparelhado com um destino personalizado.

     .. code-block:: cmake

        add_custom_command(
          OUTPUT
            ${PROJECT_BINARY_DIR}/generated/_pyaccount.c
          COMMAND
            ${Python_EXECUTABLE} ${CMAKE_CURRENT_LIST_DIR}/cffi_builder.py
          MAIN_DEPENDENCY
            ${CMAKE_CURRENT_LIST_DIR}/cffi_builder.py
          DEPENDS
            ${CMAKE_CURRENT_LIST_DIR}/account.h
          WORKING_DIRECTORY
            ${PROJECT_BINARY_DIR}/generated
          )

        add_custom_target(
          pyaccount-builder
          ALL
          DEPENDS
            ${PROJECT_BINARY_DIR}/generated/_pyaccount.c
          )

   Isso garante que o arquivo seja regenerado sempre que o cabeçalho da API for alterado.

   - Uma vez que o ``_pyaccount.c`` esteja disponível, nós o construímos 
      como um módulo Python, usando a função ``Python_add_library``, 
      fornecida no módulo ``FindPython`` do CMake.

     .. code-block:: cmake

        Python_add_library(_pyaccount
          MODULE
            account.f90
            ${PROJECT_BINARY_DIR}/generated/_pyaccount.c
          )

          # add dependency between _pyaccount target and pyaccount-builder custom target
          add_dependencies(_pyaccount pyaccount-builder)

   .. tabs::

      .. tab:: Fortran

         Um projeto base está em ``source/code/day-2/28_fortran-cffi``.
         A estrutura do código é a seguinte:

         .. code-block:: text

            28_fortran-cffi/
            ├── account
            │   ├── account.f90
            │   ├── account.h
            │   ├── cffi_builder.py
            │   ├── CMakeLists.txt
            │   ├── __init__.py
            │   └── test.py
            └── CMakeLists.txt

         Siga os prompts do ``FIXME`` em ``CMakeLists.txt`` para fazer o projeto compilar.

         #. Declare um projeto usando C e Fortran.
         #. Encontre o Python com |find_package|. 
            Solicite pelo menos a versão 3.6 com a palavra-chave 
            ``REQUIRED`` e o interpretador e os cabeçalhos de 
            desenvolvimento com a palavra-chave ``COMPONENTS``. 
            Consulte a documentação:

            .. code-block:: bash

               $ cmake --help-module FindPython | less

         #. Adicione a pasta ``account`` e ative o teste.
         #. Preencha ``CMakeLists.txt`` na pasta ``account``, seguindo os prompts do ``FIXME``.
         #. Configure, construa e execute o teste.

        Uma solução funcional está na subpasta ``solution``.

      .. tab:: C++

         Um projeto base está em  ``content/code/day-2/28_cxx-cffi``.
         A estrutura do código é a seguinte:

         .. code-block:: text

            28_fortran-cffi/
            ├── account
            │   ├── account.cpp
            │   ├── account.hpp
            │   ├── c_cpp_interface.cpp
            │   ├── account.h
            │   ├── cffi_builder.py
            │   ├── CMakeLists.txt
            │   ├── __init__.py
            │   └── test.py
            └── CMakeLists.txt

         #. Declare o projeto C++.
         #. FEncontre o Python com |find_package|. 
            Solicite pelo menos a versão 3.6 com a palavra-chave 
            ``REQUIRED`` e o interpretador e os cabeçalhos de 
            desenvolvimento com a palavra-chave ``COMPONENTS``. 
            Consulte a documentação:

            .. code-block:: bash

               $ cmake --help-module FindPython | less

         #. Adicione a pasta ``account`` e ative o teste.
         #. Preencha ``CMakeLists.txt`` na pasta ``account``, seguindo os prompts do ``FIXME``.
         #. Configure, construa e execute o teste.

         A estrutura do código é a seguinte:


.. keypoints:: Resumo

   - O CMake pode simplificar o sistema de compilação para projetos complexos e multilíngues.
