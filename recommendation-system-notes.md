# 推荐系统 / 生成式推荐知识笔记

> 用于集中记录推荐系统、生成式推荐、Semantic ID、模型评测、论文阅读和工程实践中的关键知识点。

---

## 目录

- [Codebook Collapse](#codebook-collapse)

---

## Codebook Collapse

### 一句话解释

Codebook collapse 指的是：明明设计了一个很大的 codebook，比如 1024 个 token/code，但训练后只有少数 code 被大量使用，其他大部分 code 几乎不用。

### 解决什么问题

它用于判断离散 tokenizer / Semantic ID 空间是否健康。一个好的 codebook 应该让 token 被较充分、较均衡地使用；如果大量 item 都挤到少数 token 上，说明离散空间没有被有效利用，会影响召回覆盖、token 预测、beam search 和长尾泛化。

### 直观例子

假设每层 SID codebook size 是 1024。

理想情况：

```text
1024 个 token 大多数都被使用，每个 token 下 feed 数量相对均衡，不同 token 表示不同语义区域。
```

collapse 情况：

```text
前 20 个 token 吃掉 80% feed，剩下大量 token 几乎为空。
例如 token0=17 下有 3000 万个 feed，而 token0=902 下只有 0 个 feed。
```

### 常见指标 / 判断方法

1. **code usage**：实际使用 token 数 / 总 token 数。
2. **token 分布熵**：熵越低，说明越集中在少数 token。
3. **top token 占比**：例如 top10 token 覆盖多少 feed。
4. **每个 token 下 feed 数分布**：P50 / P90 / P99 / max / 空 token 数。
5. **完整 SID path support**：统计 token0、token0-token1、token0-token1-token2 各层路径下的 feed 数分布。
6. **分桶看 hitrate**：高频 token、低频 token、空洞 token 附近的 token hit / path hit 是否差异明显。
