# DM-PFL

## Overview

This repo accompanies the paper "DM-PFL: Hitchhiking Generic Federated Learning for Efficient Shift-Robust Personalization".

The abstract from the paper is the following:

*Personalized federated learning collaboratively trains client-specific models, which holds potential for various mobile and IoT applications with heterogeneous data. However, existing solutions are vulnerable to distribution shifts between training and test data, and involve high training workloads on local devices. These two shortcomings hinder the practical usage of personalized federated learning on real-world mobile applications. To overcome these drawbacks, we explore efficient shift-robust personalization for federated learning. The principle is to hitchhike the global model to improve the shift-robustness of personalized models with minimal extra training overhead. To this end, we present DM-PFL, a novel framework that utilizes a dual masking mechanism to train both global and personalized models with weight-level parameter sharing and end-to-end sparse training. Evaluations on various datasets show that our methods not only improve the test accuracy in presence of test-time distribution shifts but also save the communication and computation costs compared to state-of-the-art personalized federated learning schemes.*

## Further Discussions

Due to the length limitations of the paper, certain contents were not included but will be provided here as an extension of our appendix.

### Further Discussions on Related Works

There are few works that also considers distribution shifts in Federated Learning that we'd like to discuss.

The paper titled "Tackling Distribution Shifts in Federated Learning with Superquantile Aggregation" aims to enhance the generalization of Generic FL when dealing with tail clients whose data distribution significantly differs from the majority. On the other hand, DM-PFL seeks to improve the shift-robustness of Personalized FL by leveraging the Generic FL model.

In the paper "Exploiting Personalized Invariance for Better Out-of-distribution Generalization in Federated Learning", they state that existing PFL methods can be viewed as "global-regularized" and their method is better because it is "dual-regularized". Following their context, DM-PFL's training method can be viewed as the global model and personalized models regularizing each other. This is because DM-PFL dynamically shares part of the weights between the global and personalized models and allows flexible transitions between the global and personalized weights.

A more recent work, titled "Test-Time Robust Personalization for Federated Learning", also utilizes the Generic FL framework to improve the shift-robustness of Personalized FL, sharing similar motivations. However, unlike our approach, they employ a two-head structure similar to Fed-RoD in the paper "On Bridging Generic and Personalized Federated Learning for Image Classification". During inference, they perform an adaptive ensemble of the two-head's outputs. In contrast, DM-PFL focuses primarily on the training procedure and addresses efficiency problems.

...

### Further Discussions on Design Details

#### How DM-PFL can achieve better shift robustness?

- Leverage Generic FL to enhance the robustness of Personalized FL (with minimal extra training overhead): DM-PFL can efficiently learn how to use the weights of the global model to improve the robustness of the personalized model. This is implemented via a dual-masking scheme and a dual-masked sparse training algorithm, with fine-grained parameter sharing to allow more flexible and learnable sharing between the global and the personalized model. The main intuition is that the global model trained via Generci FL is more robust to distribution shifts than personalized models trained via Personalized FL. This is because the global model is trained on the aggregated dataset (data across all clients), and it is likely to perform well on the shifted data at a specific client if the distribution shift is seen in the local training datasets of other clients in the federation. 
- The improved shift-robustness of global model: Although the main motivation is to enhance the robustness of the personalized model, the global model's generalization ability could also be improved(although not that consistently) by DM-PFL's training procedure.
  - As our training algorithm proceeds, the global model and personalized models are trained interdependently, with personalized models inheriting different subparts of the global model. Consequently, our algorithm enforces that the global model's weights work well even when certain weights are dropped by the personalized models, training the global model to provide more robust weights (like dropout). The fact that different subsets of the global model's weights are inherited by the personalized models has the additional benefit of encouraging the global model to provide more robust weights, akin to a form of dropout or regularization.
  - It is also worth mentioning that our fine-grained masking scheme and sparse training algorithm facilitate the personalized models to constantly explore new parameters, resulting in an *in-time over-parameterization effect*("Do We Actually Need Dense Over-Parameterization? In-Time Over-Parameterization in Sparse Training"), which have the potential to improves the robustness of the (sparse trained) models.

#### Design choice of Dual-Masking?

$$
      \theta_{g} = w_{g} \odot m_{g}, \,
        \theta_{c} = w_{g} \odot (m_{g} \cap m_{c}) + w_c \odot (m_{c}-m_{g})
$$

- Weight-level Parameter Sharing: Most existing personalized FL algorithms manually configure what parameters to share between the global and personalized model at either the model- or layer-level. Such parameter sharing schemes may be inflexible for dealing with distribution shifts. In response, under the Dual-Masking scheme, DM-PFL can learn what to share between the global and the personalized model at the weight-level for more shift-robust personalization under the control of Dual Masks.
- Both Masks and Weights Differ across Clients:  It is feasible to employ only different mask positions with the same weight parameters *i.e.,* $w \odot m_{c}$ or only different weighs at the same mask positions *i.e.,*  $(w+w_c)\odot m$ for each client. However, these schemes impose restrictions on the various personalized models across clients and are fit for heavily over-parameterized models. Training different masks only may also incur large overhead due to dependence on the lottery ticket hypothesis.

#### Why the cosine similarity of the outputs will be lower when encountering shifted input during adaptive inference?

- The intuition is that the shifted input affects the global and the personalized models differently because the global model is more shift-robust. When the input is not severely shifted, both the global and personalized models can perform well and thus they produce similar output logits that are close to the ground truth. When encountering severely shifted input, the personalized model may be more susceptible to the shift, resulting in larger discrepancies between the output logits of the global and personalized models, and thus lower cosine similarity.
- Empirical results also support this intuition.
- Also, adaptive inference is mainly for the completeness of our solution, and is not our main focus. One could apply more advanced strategies, such as model ensemble or out-of-distribution detection, for more accurate test-time shift adaptation. 

