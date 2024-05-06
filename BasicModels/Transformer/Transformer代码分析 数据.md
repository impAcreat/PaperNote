# Transformer代码分析: 数据

## main：

（以run为主分析）

### Load Data：

* 将 `.json` 文件转为 `MTDataset`（加载数据）：

```python
	train_dataset = MTDataset(config.train_data_path)
    dev_dataset = MTDataset(config.dev_data_path)
    test_dataset = MTDataset(config.test_data_path)
```



* 使用```torch.utils.data.dataloader```将上一步的```MTDataset```转为`dataloader`：

  > *在dataloader按照batch进行取数据的时候, 是取出大小等同于batch size的index列表; 然后将列表列表中的index输入到dataset的getitem()函数中,取出该index对应的数据; 最后, 对每个index对应的数据进行堆叠, 就形成了一个batch的数据*

  ```python
  train_dataloader = DataLoader(train_dataset, shuffle=True, 									batch_size=config.batch_size, collate_fn=train_dataset.collate_fn)
  dev_dataloader = DataLoader(dev_dataset, shuffle=False, 									batch_size=config.batch_size, collate_fn=dev_dataset.collate_fn)
  test_dataloader = DataLoader(test_dataset, shuffle=False, 									batch_size=config.batch_size, collate_fn=test_dataset.collate_fn)
  
  ```
  
  * 参数解释：
  * dataset：使用上一步得到的`MTDataset`

  * shuffle：是否洗牌（将数据顺序打乱）
  
  * collate_fn：手动地将抽取地样本进行堆叠
  
    * 首先`tokenize`dataset，将src_text, tgt_text转化为src_tokens, tgt_tokens
  
    ```python
    src_tokens = [[self.BOS] + self.sp_eng.EncodeAsIds(sent) + [self.EOS] 
                  for sent in src_text]
    tgt_tokens = [[self.BOS] + self.sp_chn.EncodeAsIds(sent) + [self.EOS] 
                  for sent in tgt_text]
    ```
  
    * 再进行`padding`：
  
    ```python
    batch_input = pad_sequence([torch.LongTensor(np.array(l_)) for l_ in src_tokens],
                                       batch_first=True, padding_value=self.PAD)
    batch_target = pad_sequence([torch.LongTensor(np.array(l_)) for l_ in tgt_tokens],
                                        batch_first=True, padding_value=self.PAD)
    
    ```
  
    * 最后返回`Batch`：
  
    ```python
    return Batch(src_text, tgt_text, batch_input, batch_target, self.PAD)
    ```
  
    

### 初始化模型：

```python
model = make_model(config.src_vocab_size, config.tgt_vocab_size, config.n_layers,
                       config.d_model, config.d_ff, config.n_heads, config.dropout)
model_par = torch.nn.DataParallel(model)
```

****

### criterion：

* 用于计算loss：计算loss的标准

```python
if config.use_smoothing:
        criterion = LabelSmoothing(size=config.tgt_vocab_size, padding_idx=config.padding_idx, smoothing=0.1)
        criterion.cuda()
else:
    criterion = torch.nn.CrossEntropyLoss(ignore_index=0, reduction='sum')
```

* **Label Smoothing**（未使用）：
  * 正则化方法：增加噪声，防止过拟合

****

### optimizer：

* 优化器

```python
if config.use_noamopt:
        optimizer = get_std_opt(model)
else:
    optimizer = torch.optim.AdamW(model.parameters(), lr=config.lr)
```

* **Noamopt**（使用）:

  * 学习率调整策略：warmup & 逆平方根

    * **warmup**：大型模型在训练开始时往往不稳定，较大学习率造成收敛难度大，故开始时通过warmup使用较小的学习率

  * 公式：

    ```lr = scale_factor * (model_dim ** -0.5)```

    ​	``` * min(step_num ** -0.5, step_num * warmup_steps ** -1.5)```

    *（通过min在开始时warmup）*

  * 基于Adam

****

### train:

```python
train(train_dataloader, dev_dataloader, model, model_par, criterion, optimizer)
```



## Train:

* `model.train()`与`model.eval()`：切换模式
  * 在eval模式下，不改变参数
* 以**BLEU**为标准评价训练模型（在evaluate中实现）
  * 对batch中的src进行解码，得到：decode_result
  * 用data_loader中相同的tokenizer将向量转为文本，得到：translation
  * 用sacrebleu比较translation与target_text，得到：bleu

```python
 sp_chn = chinese_tokenizer_load()
    trg = []
    res = []
    with torch.no_grad():
        # 在data的英文数据长度上遍历下标
        for batch in tqdm(data):
            # 对应的中文句子
            cn_sent = batch.trg_text
            src = batch.src
            src_mask = (src != 0).unsqueeze(-2)
            if use_beam:
                decode_result, _ = beam_search(model, src, src_mask, config.max_len,
                                               config.padding_idx, config.bos_idx, 
                                               config.eos_idx,
                                               config.beam_size, config.device)
            else:
                decode_result = batch_greedy_decode(model, src, src_mask,
                                                    max_len=config.max_len)
            decode_result = [h[0] for h in decode_result]
            translation = [sp_chn.decode_ids(_s) for _s in decode_result]
            trg.extend(cn_sent)
            res.extend(translation)
     trg = [trg]
    bleu = sacrebleu.corpus_bleu(res, trg, tokenize='zh')
    return float(bleu.score)
  
```



```python
```





