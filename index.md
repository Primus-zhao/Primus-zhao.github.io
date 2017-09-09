# Pixel Recursive Super Resolution 分析

Transfer data from low resolution to high resolution is a difficult technical problem. The reason is that low resolution figure usually means low dimension data, vice versa. To transfer data from low dim to high dim, we always need to introduce some noise (like Gaussian) to disentangle the potential corelation between these data.


# show , attend and tell model 分析

最近在学习show and tell模型，这个模型主要有两个版本，这两我们首先分析Bengio团队提出的有attention model的版本。这个版本是2015年的论文中提出，在分析之后我们也会对image caption当前比较流行的模型进行介绍和分析。
