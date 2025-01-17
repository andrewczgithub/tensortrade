# Code Structure

The TensorTrade library is modular. The `tensortrade` library usually has a common setup:

1. An abstract `MetaABC` class that highlights the methods that will generally be called inside of the main `TradingEnvironment`.
2. Specific applications of that abstract class are then specified later to make more detailed specifications.

## Example of Structure:

A good example of this structure is the `InstrumentExchange` component. It represents all exchange interactions.

The beginning of the code in [InstrumentExchange](https://github.com/notadamking/tensortrade/blob/master/tensortrade/exchanges/instrument_exchange.py) is seen here.

```py
class InstrumentExchange(object, metaclass=ABCMeta):
    """An abstract instrument exchange for use within a trading environment."""

    def __init__(self, base_instrument: str = 'USD', dtype: TypeString = np.float16, feature_pipeline: FeaturePipeline = None):
        """
        Arguments:
            base_instrument: The exchange symbol of the instrument to store/measure value in.
            dtype: A type or str corresponding to the dtype of the `observation_space`.
            feature_pipeline: A pipeline of feature transformations for transforming observations.
        """
        self._base_instrument = base_instrument
        self._dtype = dtype
        self._feature_pipeline = feature_pipeline
```

As you can see above, the [InstrumentExchange](https://github.com/notadamking/tensortrade/blob/master/tensortrade/exchanges/instrument_exchange.py) has a large majority of the instantiation details that carries over to all other reprentations of that type of class. `ABCMeta` represents that all classes that inherit it shall be recognizable as an instance of `InstrumentExchange`. This is nice when you need to do type checking.

When creating a new exchange type (everything that's an inheritance of the `InstrumentExchange`), one needs to add further details for how information should be declared by default. Once you create a new type of exchange, you can have new rules placed in by default. Let's look at the SimulatedExchange and it can have parameters dynamically set via the `**kwargs` arguement in later exchanges.

**SimulatedExchange:**

```py
class SimulatedExchange(InstrumentExchange):
    """An instrument exchange, in which the price history is based off the supplied data frame and
    trade execution is largely decided by the designated slippage model.
    If the `data_frame` parameter is not supplied upon initialization, it must be set before
    the exchange can be used within a trading environment.
    """

    def __init__(self, data_frame: pd.DataFrame = None, **kwargs):
        super().__init__(base_instrument=kwargs.get('base_instrument', 'USD'),
                         dtype=kwargs.get('dtype', np.float16),
                         feature_pipeline=kwargs.get('feature_pipeline', None))
        self._previously_transformed = False
        self._should_pretransform_obs = kwargs.get('should_pretransform_obs', False)

        if data_frame is not None:
            self.data_frame = data_frame.astype(self._dtype)

        self._commission_percent = kwargs.get('commission_percent', 0.3)
        self._base_precision = kwargs.get('base_precision', 2)
        self._instrument_precision = kwargs.get('instrument_precision', 8)
        self._initial_balance = kwargs.get('initial_balance', 1E4)
        self._min_order_amount = kwargs.get('min_order_amount', 1E-3)
        self._window_size = kwargs.get('window_size', 1)

        self._min_trade_price = kwargs.get('min_trade_price', 1E-6)
        self._max_trade_price = kwargs.get('max_trade_price', 1E6)
        self._min_trade_amount = kwargs.get('min_trade_amount', 1E-3)
        self._max_trade_amount = kwargs.get('max_trade_amount', 1E6)

        max_allowed_slippage_percent = kwargs.get('max_allowed_slippage_percent', 1.0)

        SlippageModelClass = kwargs.get('slippage_model', RandomUniformSlippageModel)
        self._slippage_model = SlippageModelClass(max_allowed_slippage_percent)
```

Everything that inherits `SimulatedExchange` uses the specified kwargs to set the parameters.

Therefore, even when we don't directly see the parameters inside of `FBMExchange`, all of the defaults are being called.

**An example:**

```py
exchange = FBMExchange(base_instrument='BTC', timeframe='1h', base_precision=4) # we're replacing the default base precision.
```
