
.. DO NOT EDIT.
.. THIS FILE WAS AUTOMATICALLY GENERATED BY SPHINX-GALLERY.
.. TO MAKE CHANGES, EDIT THE SOURCE PYTHON FILE:
.. "examples/plot_notify_failures_hyperparameter_search.py"
.. LINE NUMBERS ARE GIVEN BELOW.

.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_examples_plot_notify_failures_hyperparameter_search.py>`
        to download the full example code

.. rst-class:: sphx-glr-example-title

.. _sphx_glr_examples_plot_notify_failures_hyperparameter_search.py:


Notify Failures in Hyperparameter optimization 
==============================================

**Author(s)**: Romain Egele.

This example demonstrates how to handle failure of objectives in hyperparameter search. In many cases such as software auto-tuning (where we minimize the run-time of a software application) some configurations can create run-time errors and therefore no scalar objective is returned. A default choice could be to return in this case the worst case objective if known and it can be done inside the ``run``-function. Other possibilites are to ignore these configurations or to replace them with the running mean/min objective. To illustrate such a use-case we define an artificial ``run``-function which will fail when one of its input parameters is greater than 0.5. To define a failure, it is possible to return a "string" value with ``"F"`` as prefix such as:

.. GENERATED FROM PYTHON SOURCE LINES 10-19

.. code-block:: default



    def run(config: dict) -> float:
        if config["y"] > 0.5:
            return "F_postfix"
        else:
            return config["x"]









.. GENERATED FROM PYTHON SOURCE LINES 20-21

Then, we define the corresponding hyperparameter problem where ``x`` is the value to maximize and ``y`` is a value impact the appearance of failures.

.. GENERATED FROM PYTHON SOURCE LINES 21-30

.. code-block:: default

    from deephyper.problem import HpProblem

    problem = HpProblem()
    problem.add_hyperparameter([1, 2, 4, 8, 16, 32], "x")
    problem.add_hyperparameter((0.0, 1.0), "y")

    print(problem)






.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    Configuration space object:
      Hyperparameters:
        x, Type: Ordinal, Sequence: {1, 2, 4, 8, 16, 32}, Default: 1
        y, Type: UniformFloat, Range: [0.0, 1.0], Default: 0.5





.. GENERATED FROM PYTHON SOURCE LINES 31-32

Then, we define a centralized Bayesian optimization (CBO) search (i.e., master-worker architecture) which uses the Random-Forest regressor as default surrogate model. We will compare the ``ignore`` strategy which filters-out failed configurations, the ``mean`` strategy which replaces a failure by the running mean of collected objectives and the ``min`` strategy which replaces by the running min of collected objectives.

.. GENERATED FROM PYTHON SOURCE LINES 32-53

.. code-block:: default

    from deephyper.search.hps import CBO
    from deephyper.evaluator import Evaluator
    from deephyper.evaluator.callback import TqdmCallback

    results = {}
    max_evals = 30
    for failure_strategy in ["ignore", "mean", "min"]:
        # for failure_strategy in ["min"]:
        print(f"Executing failure strategy: {failure_strategy}")
        evaluator = Evaluator.create(
            run, method="serial", method_kwargs={"callbacks": [TqdmCallback(max_evals)]}
        )
        search = CBO(
            problem,
            evaluator,
            filter_failures=failure_strategy,
            log_dir=f"search_{failure_strategy}",
            random_state=42,
        )
        results[failure_strategy] = search.search(max_evals)





.. rst-class:: sphx-glr-script-out

 Out:

 .. code-block:: none

    Executing failure strategy: ignore

      0%|          | 0/30 [00:00<?, ?it/s]
      3%|3         | 1/30 [00:00<00:00, 115.63it/s, objective=None]
      7%|6         | 2/30 [00:00<00:00, 100.03it/s, objective=16]  
     10%|#         | 3/30 [00:00<00:00, 95.91it/s, objective=16] 
     13%|#3        | 4/30 [00:00<00:00, 93.57it/s, objective=16]
     17%|#6        | 5/30 [00:00<00:00, 92.89it/s, objective=32]
     20%|##        | 6/30 [00:00<00:00, 92.23it/s, objective=32]
     23%|##3       | 7/30 [00:00<00:00, 50.17it/s, objective=32]
     23%|##3       | 7/30 [00:00<00:00, 50.17it/s, objective=32]
     27%|##6       | 8/30 [00:00<00:00, 50.17it/s, objective=32]
     30%|###       | 9/30 [00:00<00:00, 50.17it/s, objective=32]
     33%|###3      | 10/30 [00:00<00:00, 50.17it/s, objective=32]
     37%|###6      | 11/30 [00:00<00:00, 50.17it/s, objective=32]
     40%|####      | 12/30 [00:00<00:00, 50.17it/s, objective=32]
     43%|####3     | 13/30 [00:00<00:00, 47.45it/s, objective=32]
     43%|####3     | 13/30 [00:00<00:00, 47.45it/s, objective=32]
     47%|####6     | 14/30 [00:00<00:00, 47.45it/s, objective=32]
     50%|#####     | 15/30 [00:00<00:00, 47.45it/s, objective=32]
     53%|#####3    | 16/30 [00:00<00:00, 47.45it/s, objective=32]
     57%|#####6    | 17/30 [00:00<00:00, 47.45it/s, objective=32]
     60%|######    | 18/30 [00:00<00:00, 20.45it/s, objective=32]
     60%|######    | 18/30 [00:00<00:00, 20.45it/s, objective=32]
     63%|######3   | 19/30 [00:00<00:00, 20.45it/s, objective=32]
     67%|######6   | 20/30 [00:00<00:00, 20.45it/s, objective=32]
     70%|#######   | 21/30 [00:00<00:00, 20.45it/s, objective=32]
     73%|#######3  | 22/30 [00:00<00:00, 20.45it/s, objective=32]
     77%|#######6  | 23/30 [00:00<00:00, 20.45it/s, objective=32]
     80%|########  | 24/30 [00:00<00:00, 20.45it/s, objective=32]
     83%|########3 | 25/30 [00:00<00:00, 20.45it/s, objective=32]
     87%|########6 | 26/30 [00:00<00:00, 20.45it/s, objective=32]
     90%|######### | 27/30 [00:00<00:00, 20.45it/s, objective=32]
     93%|#########3| 28/30 [00:00<00:00, 20.45it/s, objective=32]
     97%|#########6| 29/30 [00:00<00:00, 20.45it/s, objective=32]
    100%|##########| 30/30 [00:00<00:00, 20.45it/s, objective=32]Executing failure strategy: mean


      0%|          | 0/30 [00:00<?, ?it/s]    100%|##########| 30/30 [00:00<00:00, 40.87it/s, objective=32]


      3%|3         | 1/30 [00:00<00:02, 13.83it/s, objective=None]

      7%|6         | 2/30 [00:00<00:01, 23.90it/s, objective=16]  

     10%|#         | 3/30 [00:00<00:00, 31.63it/s, objective=16]

     13%|#3        | 4/30 [00:00<00:00, 37.78it/s, objective=16]

     13%|#3        | 4/30 [00:00<00:00, 37.78it/s, objective=16]

     17%|#6        | 5/30 [00:00<00:00, 37.78it/s, objective=32]

     20%|##        | 6/30 [00:00<00:00, 37.78it/s, objective=32]

     23%|##3       | 7/30 [00:00<00:00, 37.78it/s, objective=32]

     27%|##6       | 8/30 [00:00<00:00, 37.14it/s, objective=32]

     27%|##6       | 8/30 [00:00<00:00, 37.14it/s, objective=32]

     30%|###       | 9/30 [00:00<00:00, 37.14it/s, objective=32]

     33%|###3      | 10/30 [00:00<00:00, 37.14it/s, objective=32]

     37%|###6      | 11/30 [00:00<00:00, 37.14it/s, objective=32]

     40%|####      | 12/30 [00:00<00:01, 13.69it/s, objective=32]

     40%|####      | 12/30 [00:00<00:01, 13.69it/s, objective=32]

     43%|####3     | 13/30 [00:00<00:01, 13.69it/s, objective=32]

     47%|####6     | 14/30 [00:01<00:01, 13.69it/s, objective=32]

     50%|#####     | 15/30 [00:01<00:01,  8.03it/s, objective=32]

     50%|#####     | 15/30 [00:01<00:01,  8.03it/s, objective=32]

     53%|#####3    | 16/30 [00:01<00:01,  8.03it/s, objective=32]

     57%|#####6    | 17/30 [00:01<00:02,  6.36it/s, objective=32]

     57%|#####6    | 17/30 [00:01<00:02,  6.36it/s, objective=32]

     60%|######    | 18/30 [00:02<00:01,  6.36it/s, objective=32]

     63%|######3   | 19/30 [00:02<00:01,  5.51it/s, objective=32]

     63%|######3   | 19/30 [00:02<00:01,  5.51it/s, objective=32]

     67%|######6   | 20/30 [00:02<00:01,  5.34it/s, objective=32]

     67%|######6   | 20/30 [00:02<00:01,  5.34it/s, objective=32]

     70%|#######   | 21/30 [00:02<00:01,  5.15it/s, objective=32]

     70%|#######   | 21/30 [00:02<00:01,  5.15it/s, objective=32]

     73%|#######3  | 22/30 [00:03<00:01,  4.97it/s, objective=32]

     73%|#######3  | 22/30 [00:03<00:01,  4.97it/s, objective=32]

     77%|#######6  | 23/30 [00:03<00:01,  4.52it/s, objective=32]

     77%|#######6  | 23/30 [00:03<00:01,  4.52it/s, objective=32]

     80%|########  | 24/30 [00:03<00:01,  4.51it/s, objective=32]

     80%|########  | 24/30 [00:03<00:01,  4.51it/s, objective=32]

     83%|########3 | 25/30 [00:03<00:01,  4.48it/s, objective=32]

     83%|########3 | 25/30 [00:03<00:01,  4.48it/s, objective=32]

     87%|########6 | 26/30 [00:04<00:00,  4.12it/s, objective=32]

     87%|########6 | 26/30 [00:04<00:00,  4.12it/s, objective=32]

     90%|######### | 27/30 [00:04<00:00,  4.18it/s, objective=32]

     90%|######### | 27/30 [00:04<00:00,  4.18it/s, objective=32]

     93%|#########3| 28/30 [00:04<00:00,  4.22it/s, objective=32]

     93%|#########3| 28/30 [00:04<00:00,  4.22it/s, objective=32]

     97%|#########6| 29/30 [00:04<00:00,  3.92it/s, objective=32]

     97%|#########6| 29/30 [00:04<00:00,  3.92it/s, objective=32]

    100%|##########| 30/30 [00:05<00:00,  4.05it/s, objective=32]

    100%|##########| 30/30 [00:05<00:00,  4.05it/s, objective=32]Executing failure strategy: min

      0%|          | 0/30 [00:00<?, ?it/s]    100%|##########| 30/30 [00:05<00:00,  5.83it/s, objective=32]

      3%|3         | 1/30 [00:00<00:00, 112.65it/s, objective=None]
      7%|6         | 2/30 [00:00<00:00, 99.76it/s, objective=16]   
     10%|#         | 3/30 [00:00<00:00, 96.69it/s, objective=16]
     13%|#3        | 4/30 [00:00<00:00, 95.57it/s, objective=16]
     17%|#6        | 5/30 [00:00<00:00, 43.04it/s, objective=16]
     17%|#6        | 5/30 [00:00<00:00, 43.04it/s, objective=32]
     20%|##        | 6/30 [00:00<00:00, 43.04it/s, objective=32]
     23%|##3       | 7/30 [00:00<00:00, 43.04it/s, objective=32]
     27%|##6       | 8/30 [00:00<00:00, 43.04it/s, objective=32]
     30%|###       | 9/30 [00:00<00:00, 43.04it/s, objective=32]
     33%|###3      | 10/30 [00:00<00:00, 43.04it/s, objective=32]
     37%|###6      | 11/30 [00:00<00:00, 23.13it/s, objective=32]
     37%|###6      | 11/30 [00:00<00:00, 23.13it/s, objective=32]
     40%|####      | 12/30 [00:00<00:00, 23.13it/s, objective=32]
     43%|####3     | 13/30 [00:00<00:00, 23.13it/s, objective=32]
     47%|####6     | 14/30 [00:01<00:01, 10.02it/s, objective=32]
     47%|####6     | 14/30 [00:01<00:01, 10.02it/s, objective=32]
     50%|#####     | 15/30 [00:01<00:01, 10.02it/s, objective=32]
     53%|#####3    | 16/30 [00:01<00:01,  7.33it/s, objective=32]
     53%|#####3    | 16/30 [00:01<00:01,  7.33it/s, objective=32]
     57%|#####6    | 17/30 [00:01<00:01,  7.33it/s, objective=32]
     60%|######    | 18/30 [00:02<00:02,  5.99it/s, objective=32]
     60%|######    | 18/30 [00:02<00:02,  5.99it/s, objective=32]
     63%|######3   | 19/30 [00:02<00:01,  5.68it/s, objective=32]
     63%|######3   | 19/30 [00:02<00:01,  5.68it/s, objective=32]
     67%|######6   | 20/30 [00:02<00:01,  5.39it/s, objective=32]
     67%|######6   | 20/30 [00:02<00:01,  5.39it/s, objective=32]
     70%|#######   | 21/30 [00:02<00:01,  4.80it/s, objective=32]
     70%|#######   | 21/30 [00:02<00:01,  4.80it/s, objective=32]
     73%|#######3  | 22/30 [00:03<00:01,  4.71it/s, objective=32]
     73%|#######3  | 22/30 [00:03<00:01,  4.71it/s, objective=32]
     77%|#######6  | 23/30 [00:03<00:01,  4.63it/s, objective=32]
     77%|#######6  | 23/30 [00:03<00:01,  4.63it/s, objective=32]
     80%|########  | 24/30 [00:03<00:01,  4.56it/s, objective=32]
     80%|########  | 24/30 [00:03<00:01,  4.56it/s, objective=32]
     83%|########3 | 25/30 [00:03<00:01,  4.17it/s, objective=32]
     83%|########3 | 25/30 [00:03<00:01,  4.17it/s, objective=32]
     87%|########6 | 26/30 [00:04<00:00,  4.23it/s, objective=32]
     87%|########6 | 26/30 [00:04<00:00,  4.23it/s, objective=32]
     90%|######### | 27/30 [00:04<00:00,  4.26it/s, objective=32]
     90%|######### | 27/30 [00:04<00:00,  4.26it/s, objective=32]
     93%|#########3| 28/30 [00:04<00:00,  3.93it/s, objective=32]
     93%|#########3| 28/30 [00:04<00:00,  3.93it/s, objective=32]
     97%|#########6| 29/30 [00:04<00:00,  4.01it/s, objective=32]
     97%|#########6| 29/30 [00:04<00:00,  4.01it/s, objective=32]
    100%|##########| 30/30 [00:05<00:00,  4.05it/s, objective=32]
    100%|##########| 30/30 [00:05<00:00,  4.05it/s, objective=32]



.. GENERATED FROM PYTHON SOURCE LINES 54-55

Finally we plot the collected results

.. GENERATED FROM PYTHON SOURCE LINES 55-75

.. code-block:: default

    import matplotlib.pyplot as plt
    import numpy as np

    plt.figure()

    for i, (failure_strategy, df) in enumerate(results.items()):
        plt.subplot(3, 1, i + 1)
        if df.objective.dtype != np.float64:
            x = np.arange(len(df))
            mask_failed = np.where(df.objective.str.startswith("F"))[0]
            mask_success = np.where(~df.objective.str.startswith("F"))[0]
            x_success, x_failed = x[mask_success], x[mask_failed]
            y_success = df["objective"][mask_success].astype(float)
        plt.scatter(x_success, y_success, label=failure_strategy)
        plt.scatter(x_failed, np.zeros(x_failed.shape), marker="v", color="red")

        plt.xlabel(r"Iterations")
        plt.ylabel(r"Objective")
        plt.legend()
    plt.show()



.. image-sg:: /examples/images/sphx_glr_plot_notify_failures_hyperparameter_search_001.png
   :alt: plot notify failures hyperparameter search
   :srcset: /examples/images/sphx_glr_plot_notify_failures_hyperparameter_search_001.png
   :class: sphx-glr-single-img






.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  11.158 seconds)


.. _sphx_glr_download_examples_plot_notify_failures_hyperparameter_search.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: plot_notify_failures_hyperparameter_search.py <plot_notify_failures_hyperparameter_search.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: plot_notify_failures_hyperparameter_search.ipynb <plot_notify_failures_hyperparameter_search.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
