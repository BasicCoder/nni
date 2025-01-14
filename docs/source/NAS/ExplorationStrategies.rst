Exploration Strategies for Multi-trial NAS
==========================================

Usage of Exploration Strategy
-----------------------------

To use an exploration strategy, users simply instantiate an exploration strategy and pass the instantiated object to ``RetiariiExperiment``. Below is a simple example.

.. code-block:: python

  import nni.retiarii.strategy as strategy

  exploration_strategy = strategy.Random(dedup=True)  # dedup=False if deduplication is not wanted

Supported Exploration Strategies
--------------------------------

NNI provides the following exploration strategies for multi-trial NAS.

.. list-table::
   :header-rows: 1
   :widths: auto

   * - Name
     - Brief Introduction of Algorithm
   * - `Random Strategy <./ApiReference.rst#nni.retiarii.strategy.Random>`__
     - Randomly sampling new model(s) from user defined model space. (``nni.retiarii.strategy.Random``)
   * - `Grid Search <./ApiReference.rst#nni.retiarii.strategy.GridSearch>`__
     - Sampling new model(s) from user defined model space using grid search algorithm. (``nni.retiarii.strategy.GridSearch``)
   * - `Regularized Evolution <./ApiReference.rst#nni.retiarii.strategy.RegularizedEvolution>`__
     - Generating new model(s) from generated models using `regularized evolution algorithm <https://arxiv.org/abs/1802.01548>`__ . (``nni.retiarii.strategy.RegularizedEvolution``)
   * - `TPE Strategy <./ApiReference.rst#nni.retiarii.strategy.TPEStrategy>`__
     - Sampling new model(s) from user defined model space using `TPE algorithm <https://papers.nips.cc/paper/2011/file/86e8f7ab32cfd12577bc2619bc635690-Paper.pdf>`__ . (``nni.retiarii.strategy.TPEStrategy``)
   * - `RL Strategy <./ApiReference.rst#nni.retiarii.strategy.PolicyBasedRL>`__
     - It uses `PPO algorithm <https://arxiv.org/abs/1707.06347>`__ to sample new model(s) from user defined model space. (``nni.retiarii.strategy.PolicyBasedRL``)

Customize Exploration Strategy
------------------------------

If users want to innovate a new exploration strategy, they can easily customize a new one following the interface provided by NNI. Specifically, users should inherit the base strategy class ``BaseStrategy``, then implement the member function ``run``. This member function takes ``base_model`` and ``applied_mutators`` as its input arguments. It can simply apply the user specified mutators in ``applied_mutators`` onto ``base_model`` to generate a new model. When a mutator is applied, it should be bound with a sampler (e.g., ``RandomSampler``). Every sampler implements the ``choice`` function which chooses value(s) from candidate values. The ``choice`` functions invoked in mutators are executed with the sampler.

Below is a very simple random strategy, which makes the choices completely random.

.. code-block:: python

    from nni.retiarii import Sampler

    class RandomSampler(Sampler):
        def choice(self, candidates, mutator, model, index):
            return random.choice(candidates)

    class RandomStrategy(BaseStrategy):
        def __init__(self):
            self.random_sampler = RandomSampler()

        def run(self, base_model, applied_mutators):
            _logger.info('stargety start...')
            while True:
                avail_resource = query_available_resources()
                if avail_resource > 0:
                    model = base_model
                    _logger.info('apply mutators...')
                    _logger.info('mutators: %s', str(applied_mutators))
                    for mutator in applied_mutators:
                        mutator.bind_sampler(self.random_sampler)
                        model = mutator.apply(model)
                    # run models
                    submit_models(model)
                else:
                    time.sleep(2)

You can find that this strategy does not know the search space beforehand, it passively makes decisions every time ``choice`` is invoked from mutators. If a strategy wants to know the whole search space before making any decision (e.g., TPE, SMAC), it can use ``dry_run`` function provided by ``Mutator`` to obtain the space. An example strategy can be found :githublink:`here <nni/retiarii/strategy/tpe_strategy.py>`.

After generating a new model, the strategy can use our provided APIs (e.g., ``submit_models``, ``is_stopped_exec``) to submit the model and get its reported results. More APIs can be found in `API References <./ApiReference.rst>`__.
