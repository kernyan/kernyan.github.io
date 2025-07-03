| method          | SM    | thread |
|-----------------|-------|--------|
| notify dispatch | 1 + 8 | 128    |
| dispatch        | 20    | 128    |
| combine         | 20    | 768    |


| method          | SM                          | thread |
|-----------------|-----------------------------|--------|
| notify dispatch | 1 + 8 (1 send, 8 receive)   | 128    |
| dispatch        | 20 (even send, odd receive) | 128    |
| combine         | 20 (even send, odd receive) | 768    |


| idea                      | description                           | location                                                                                                                                               |
|---------------------------|---------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| dual stream communication | toggling communication/compute stream | [DeepEP/csrc/deep_ep.cpp](https://github.com/deepseek-ai/DeepEP/blob/3885404ffb31a3d9cbd238aa1a366408881bb069/csrc/deep_ep.cpp#L701-L705)              |
| out-of-doc PTX load/store | by-pass L1 cache load/store           | [DeepEP/csrc/kernels/utils.cuh](https://github.com/deepseek-ai/DeepEP/blob/3885404ffb31a3d9cbd238aa1a366408881bb069/csrc/kernels/utils.cuh#L239-L243)  |
| warp specialization       | kernel do branching in same warp      | [DeepEP/csrc/kernels/intranode.cu](https://github.com/deepseek-ai/DeepEP/blob/3885404ffb31a3d9cbd238aa1a366408881bb069/csrc/kernels/intranode.cu#L239) |
| topology-aware routing | forward either on IB or NVLink | [DeepEP/deep_ep/buffer.py](https://github.com/deepseek-ai/DeepEP/blob/3885404ffb31a3d9cbd238aa1a366408881bb069/deep_ep/buffer.py#L288-L311)
