Tutorial CMake 
=======================


O CMake é uma ferramenta de construção multiplataforma, independente de linguagem, quem vem
se tornando padrão, emm grandes projeto, o qual é usando para construir, testar e
implantar suas bases de código. Neste tutorial você vai aprender:

- Escrever um sistema de compilação CMake para projetos C, C++ e Fortran produzindo
  bibliotecas e/ou executáveis.
- Executar testes para seu código com CTest.
- Certificar-se de que seu sistema de compilação funciona em diferentes plataformas.
- Detectar e usar dependências externas em seu projeto.
- Construir com segurança e eficácia projetos que usam linguagens mistas (Python+C/C++,
  Python+Fortran, Fortran+C/C++)

.. prereq::

   Antes de iniciar esse tutorial, recomenda-se ter acesso
   a um computador com compilador para sua linguagem de programação favorita 
   e uma versão recente do CMake.

   Caso não tenha ainda esse ambiente instalado, recomendamos que você configure 
   um ambiente de software isolado usando ``conda``. Para computadores com sistemas 
   Windows, recomendamos usar o Subsistema para Linux (WSL). 
   Instruções detalhadas podem ser encontradas na página :doc:`setup`.

.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: Pré-Requisitos

   setup


.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: Atividades

   hello-cmake
   cmake-syntax
   hello-ctest
   environment
   probing
   targets
   dependencies
   ..fetch-content
   ..python-bindings
   ..tips-and-tricks


.. toctree::
   :hidden:
   :maxdepth: 1
   :caption: Topicos Adicionais

   .. cxx-fortran

.. see also the schedule in guide.rst

.. csv-table::
   :widths: auto
   :delim: ;

   30 min ; :doc:`hello-cmake`
   30 min ; :doc:`cmake-syntax`
   30 min ; :doc:`hello-ctest`
   30 min ; :doc:`environment`
   30 min ; :doc:`probing`
   40 min ; :doc:`targets`
   ..40 min ; :doc:`dependencies`
   ..40 min ; :doc:`fetch-content`
   ..30 min ; :doc:`python-bindings`
   ..30 min ; :doc:`tips-and-tricks`



.. toctree::
   :maxdepth: 1
   :caption: Referências
   
   quick-reference
   zbibliography


.. _learner-personas:

Para quem este tutorial é voltado?
-----------------------------------

Este curso é voltado para estudantes que ouviram falar do `CMake`_ e querem 
aprender como usá-lo efetivamente em projetos que estão trabalhando.

Este curso não pressupõe experiência anterior com `CMake`_. Você terá que estar
familiarizado com algumas ferramentas comumente usadas para construir software
na sua linguagem de escolha: C++, C, Fortran.
Especificamente, este tutorial pressupõe que os participantes tenham alguma experiência 
anterior ou ou algum conhecimento dos seguintes tópicos:

- Compilação e vinculação de executáveis e bibliotecas.
- Diferenças entre bibliotecas compartilhadas e estáticas.
- Testes automatizados


Convenções textuais e gráficas
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Adotamos algumas convenções que ajudam a organizar o material.

Assinaturas de função
   Eles são mostrados em um bloco de texto marcado com um emoji de chave inglesa:

   .. signature:: |cmake_minimum_required|

      .. code-block:: cmake

         cmake_minimum_required(VERSION <min>[...<max>] [FATAL_ERROR])

   

Parâmetros de comando
   A descrição dos parâmetros de comando aparecerá em uma caixa de 
   texto separada. Ele será marcado com um emoji de laptop:

   .. parameters::

      ``VERSION``
          Minimum and, optionally, maximum version of CMake to use.
      ``FATAL_ERROR``
          Raise a fatal error if the version constraint is not satisfied. This
          option is ignored by CMake >=2.6

   

Type-along
   O texto e o código dessas atividades estão em uma caixa de texto separada, 
   marcada com um emoji de teclado:

   .. typealong:: Vejamos um exemplo

      .. code-block:: cmake

         cmake_minimum_required(VERSION 3.16)

         project(Hello LANGUAGES CXX)

  

Exercícios
   Cada exercício é apresentado em uma caixa de texto separada, 
   marcada com um emoji de mão. Cada caixa apresenta o exercício e descreve os objetivos:

   .. exercise:: Exercício 10

      Usaremos o CMake para construir uma biblioteca Fortran.



Veja Também
------------

Existem muitas fontes gratuitas on-line sobre o CMake:

- `Documentação CMake official
  <https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html>`_.
- `CMake tutorial <https://cmake.org/cmake/help/v3.19/guide/tutorial/index.html#guide:CMake%20Tutorial>`_.
- Curso de Treinamento do `HEP Software Foundation <https://hsf-training.github.io/hsf-training-cmake-webpage/>`_.


Você também pode consultar os seguintes livros:

- **Professional CMake: A Practical Guide** por Craig Scott.
- **CMake Cookbook** por Radovan Bast e Roberto Di Remigio que tem o  seguinte repositório  `GitHub <https://github.com/dev-cafe/cmake-cookbook>`_


Créditos
---------

Este tutorial é baseado no material desenvolvido por `EuroCC National Competence Center
Sweden (ENCCS) <https://enccs.se/>`_ e é licenciado sob a `Creative Commons
Attribution license (CC-BY-4.0)
<https://creativecommons.org/licenses/by/4.0/>`_.


Este é um resumo (e não um substituto para) a licença
Creative Commons Attribution 4.0 International (CC-BY-4.0).

Você tem o direito de:

- **Compartilhar** — copiar e redistribuir o material em qualquer suporte ou formato
- **Adaptar** — remixar, transformar, e criar a partir do material para qualquer fim, mesmo que comercial.

Esta licença é aceitável para Trabalhos Culturais Livres.

O licenciante não pode revogar estes direitos desde que você respeite os termos da licença.

De acordo com os termos seguintes:

- **Atribuição** — Você deve dar o crédito apropriado, prover um link para a licença e indicar se mudanças foram feitas. Você deve fazê-lo em qualquer circunstância razoável, mas de nenhuma maneira que sugira que o licenciante apoia você ou o seu uso.
- **Sem restrições adicionais** — Você não pode aplicar termos jurídicos ou medidas de caráter tecnológico que restrinjam legalmente outros de fazerem algo que a licença permita.


Software
^^^^^^^^
Os exemplos de código e exercícios nesta lição foram adaptados do 
GitHub `CMake Cookbook <https://github.com/dev-cafe/cmake-cookbook>`_.

