# fastdl

fastdl is the deep learning library built on the top of `pytorch` & `deepspeed` for making the your deep learning journey more smooth and fast.

## Supported features

- Single trainer for most of use-cases
- Full support to distributed training on gpus (thanks to deepspeed!)
- Ease of use & getting rid of boiler-plate code
- Automatic logging with wandb
- Early stopping and automatic saving
- Training with mixed-precision
- Gradient Accumulation

```python

from fastdl import DeepTrainer, TrainerConfig

class Trainer(TorchTrainer):

    def __init__(self, model, args):
        self.model = model
        self.lr = args.lr

        # call this at end only
        super().__init__(args)

    def configure_optimizers(self):
        return torch.optim.Adam(self.model.parameters(), lr=self.lr)

    def training_step(self, batch, batch_idx):
        # This method should look something like this

        batch = batch.to(self.device)

        out = self.model(batch)
        loss = out.mean()

        return loss

    def validation_step(self, batch):
        # This method should look something like this

        batch = batch.to(self.device)

        with torch.no_grad():
            out = self.model(batch)
            loss = out.mean()

        return loss

# define model architecture
model = .....

# define dataset
tr_dataset = .....
val_dataset = .....

# load default args
args = TrainerConfig.from_default()
# change default args as per need
args.update(
    {
        "lr": 2e-5,
        "save_dir": "wts"
    }
)

trainer = Trainer(model, args)
trainer.fit(tr_dataset, val_dataset)
```

```python

import fastdl

model = ...
optimizer = ...
args = ...

trainer = fastdl.DeepTrainer(args, model, optimizer)
trainer.fit(tr_data, val_data)
```

## Arguments

```python
"""
    base_dir :: str : root dir for any kind of saving (default = ".")
    map_location :: torch.device : argument used in torch.load() while loading model-state-dict (default = torch.device("cuda:0"))

    save_dir :: str : If specified training stuff and model weights will be saved in this dir (default = None)
    load_dir :: str : If specified training stuff and model weights will be loaded from this dir (default = None)

    project_name :: str : Project name in wandb (default = None)
    wandb_run_name :: str : run name in wandb (default = None)
    wandb_off :: bool : If you want to disable wandb; useful for testing (default = False)

    max_epochs :: int : No of epochs (default = 5)

    early_stop_n :: int : Enable early stopping by specifying how many epochs to look-up before stopping (default = None)
    save_epoch_dir :: str : If specified, ckpt will be saved at epoch level if loss decreases
"""
```

### Notes

- Currently, this can't be used with models involving multiple optimizers (like GANs).
- Don't forget to send your batch to `self.device`. Model will be automatically transferred to `self.device`.
- Model weights will be in `.pt` file while other training stuff will be in `.tar`.