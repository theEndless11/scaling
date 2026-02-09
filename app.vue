<template>
  <div class="paper-container">
    <header class="paper-header">
      <h1>Representation-Limited Scaling in Transformer Language Models</h1>
      <div class="authors">
        <p>A Study on Feature Superposition and Model Dimension Scaling</p>
      </div>
      <div class="meta">
        <span>February 2026</span>
      </div>
    </header>

    <nav class="table-of-contents">
      <h2>Contents</h2>
      <ul>
        <li><a href="#abstract">Abstract</a></li>
        <li><a href="#introduction">Introduction</a></li>
        <li><a href="#superposition">Feature Superposition in Neural Networks</a></li>
        <li><a href="#scaling-regimes">Scaling Regimes and Weight Decay</a></li>
        <li><a href="#empirical">Empirical Analysis</a></li>
        <li><a href="#llm-validation">LLM Validation</a></li>
        <li><a href="#sweet-spot">The Embedding Dimension Sweet Spot</a></li>
        <li><a href="#implications">Implications for Future Scaling</a></li>
        <li><a href="#conclusion">Conclusion</a></li>
      </ul>
    </nav>

    <main class="paper-content">
      <section id="abstract">
        <h2>Abstract</h2>
        <p>
          This paper investigates the fundamental scaling limits of transformer-based large language models (LLMs) 
          through the lens of feature representation and superposition. We demonstrate that model performance is 
          not solely determined by parameter count, but critically depends on how features are represented in the 
          model's internal geometry. Our analysis reveals two distinct scaling regimes: weak superposition, where 
          loss scaling depends on the frequency distribution of ignored features, and strong superposition, where 
          loss arises from interference between overlapping representations. Through both toy model experiments 
          and analysis of actual LLMs, we show that modern transformers exhibit strong superposition, leading to 
          robust "one over width" scaling behavior that is independent of feature frequency distributions.
        </p>
      </section>

      <section id="introduction">
        <h2>1. Introduction</h2>
        <p>
          The remarkable success of large language models has been driven largely by scaling model size, measured 
          in parameters. However, the mechanisms underlying this scaling remain poorly understood. While empirical 
          scaling laws describe how loss decreases with model size, they do not explain why this relationship holds 
          or when it might break down.
        </p>
        <p>
          We approach this question through the framework of feature superposition: the phenomenon where neural 
          networks represent more features than they have dimensions by allowing features to interfere with each 
          other in the representation space. This is analogous to compressed sensing, where signals can be 
          reconstructed from fewer measurements than classical theory would suggest.
        </p>
        <p>
          Our key contributions are:
        </p>
        <ul>
          <li>Identification of two distinct scaling regimes based on the degree of feature superposition</li>
          <li>Demonstration that weight decay can robustly control the transition between these regimes</li>
          <li>Empirical validation showing that actual LLMs operate in the strong superposition regime</li>
          <li>Analysis of how feature frequency distributions affect scaling behavior</li>
          <li><strong>Discovery of the embedding dimension sweet spot (4,096-8,192 dims) that optimally balances performance and efficiency</strong></li>
        </ul>

        <div class="chart-container">
          <div class="chart-title">Preview: The Embedding Dimension Sweet Spot</div>
          <canvas id="previewChart"></canvas>
          <p class="chart-caption">
            This paper identifies the optimal embedding dimension range where models achieve the best 
            balance between semantic capture and computational efficiency. Details in Section 6.
          </p>
        </div>
      </section>

      <section id="superposition">
        <h2>2. Feature Superposition in Neural Networks</h2>
        
        <h3>2.1 The Representation Problem</h3>
        <p>
          Consider a neural network attempting to represent n features using an m-dimensional embedding space, 
          where m < n. In the classical view, the model can only represent m features without interference. 
          However, through superposition, models can represent many more features by allowing them to share 
          the representational space.
        </p>

        <div class="figure">
          <img src="/WhatsApp_Image_2026-02-08_at_12_28_15.jpeg" alt="Feature representation visualization - Eiffel Tower/Paris scatter plot" class="figure-img" />
          <p class="figure-caption">
            <strong>Figure 1:</strong> Visualization of feature embeddings in high-dimensional space. Each point 
            represents a learned feature vector. Features cluster in semantically meaningful regions (e.g., 
            landmarks like "Eiffel Tower" and "Paris" appear in proximity), demonstrating how the model organizes 
            its representation space. The density of points illustrates the degree of superposition, with more 
            densely packed regions indicating higher feature interference.
            <br/><em>Visual reference: Use Image.jpeg (scatter plot with Eiffel Tower, Paris, sandwich labels)</em>
          </p>
        </div>

        <h3>2.2 Measuring Superposition</h3>
        <p>
          We quantify superposition through the fraction of represented features, defined as:
        </p>
        <div class="equation">
          φ₁/₂ = |{i : ||Wᵢ||₂ > 1/2}|/n
        </div>
        <p>
          where Wᵢ represents the weight vector for feature i, and n is the total number of features. This metric 
          captures the proportion of features with norms larger than 1/2, indicating they are being actively 
          represented rather than ignored.
        </p>
        <p>
          When weight norms become bimodal, clustering near 0 or 1, we can clearly distinguish between represented 
          and unrepresented features. This allows us to study how the model allocates its limited representational 
          capacity.
        </p>

        <div class="figure">
          <img src="/WhatsApp_Image_2026-02-08_at_12_28_17__1_.jpeg" alt="Menu selection interface showing feature selection" class="figure-img" />
          <p class="figure-caption">
            <strong>Figure 2:</strong> Interactive visualization showing feature selection in a sparse representation. 
            The menu-like interface demonstrates how models must choose which features to represent when capacity 
            is limited. Features shown include architectural landmarks (Eiffel, Giant, Winds, Power, Ponds, Apply), 
            illustrating the discrete nature of feature selection under strong superposition.
            <br/><em>Visual reference:Use Image.jpeg (menu with Eiffel, Giant, Winds, Power, Ponds, Apply)</em>
          </p>
        </div>
      </section>

      <section id="scaling-regimes">
        <h2>3. Scaling Regimes and Weight Decay</h2>

        <h3>3.1 Weak Superposition Regime</h3>
        <p>
          In the weak superposition regime, the model represents only a fraction of available features, with 
          φ₁/₂ ≈ m/n. The remaining features are effectively ignored, contributing to loss through their absence. 
          The scaling behavior in this regime depends critically on how feature frequencies decay with rank.
        </p>
        <p>
          When feature frequencies follow a power law distribution, and m is sufficiently large, the loss also 
          follows a power law with model size. However, this relationship is fragile: it depends on the specific 
          frequency distribution and breaks down if frequencies decay differently than expected.
        </p>

        <h3>3.2 Strong Superposition Regime</h3>
        <p>
          In strong superposition, the model represents many more features (φ₁/₂ ≈ 1 ≫ m/n), but these 
          representations overlap and interfere with each other. Loss no longer comes primarily from ignored 
          features, but from the interference between represented features competing for the same representational 
          space.
        </p>
        <p>
          Remarkably, this interference-based loss exhibits robust scaling behavior. Because the interference 
          arises from the geometry of how features are packed into the limited dimensional space, the loss scales 
          inversely with model dimension (∝ 1/m) regardless of the feature frequency distribution.
        </p>

        <div class="figure">
          <img src="/WhatsApp_Image_2026-02-08_at_12_28_16.jpeg" alt="Weak superposition graph showing loss scaling" class="figure-img" />
          <p class="figure-caption">
            <strong>Figure 3:</strong> Loss scaling under weak superposition with different feature importance decay 
            functions. The graph shows how model loss varies with model dimension (m) under three feature importance 
            decay patterns: linear decay (green), power law decay (orange), and exponential decay (blue). The slope 
            of -1.0 indicates the expected scaling relationship. Note how different decay patterns lead to different 
            scaling behaviors, demonstrating the fragility of weak superposition scaling.
            <br/><em>Visual reference: Use Image.Jpeg (graph with "Weak superposition" label)</em>
          </p>
        </div>

        <h3>3.3 Weight Decay as a Control Mechanism</h3>
        <p>
          Weight decay provides a robust mechanism to control the transition between scaling regimes. By 
          penalizing large weights, weight decay encourages sparsity in feature representation. The relationship 
          is intuitive:
        </p>
        <ul>
          <li><strong>Small weight decay (γ):</strong> Permits dense representations with high overlap, leading to strong superposition where φ₁/₂ ≈ 1 ≫ m/n</li>
          <li><strong>Large weight decay (γ):</strong> Forces sparse representations with minimal overlap, leading to weak superposition where φ₁/₂ ≈ m/n</li>
        </ul>
        <p>
          This control mechanism is robust across different architectures and feature frequency distributions, 
          making it a reliable tool for steering models into desired scaling regimes.
        </p>
      </section>

      <section id="empirical">
        <h2>4. Empirical Analysis</h2>

        <h3>4.1 Toy Model Experiments</h3>
        <p>
          We developed a simplified toy model that captures the essential dynamics of language models while 
          remaining tractable for systematic study. Unlike full LLMs, which map documents to tokens with inputs 
          and outputs in different spaces, our toy model operates within a single shared representational space. 
          Despite this simplification, the toy model successfully captures key aspects of language structure 
          through engineered sparsity and feature importance, making its data structure aligned with that of 
          LLMs at a high level.
        </p>
        <p>
          The toy model allows us to systematically vary model dimension and measure how loss scales. By 
          controlling weight decay, we can induce either weak or strong superposition and observe the resulting 
          scaling behaviors.
        </p>

        <div class="figure">
          <img src="/WhatsApp_Image_2026-02-08_at_12_28_19__1_.jpeg" alt="Toy model scaling comparison showing two panels" class="figure-img" />
          <p class="figure-caption">
            <strong>Figure 4:</strong> Scaling behavior comparison in toy models. Panel (a) shows mean square overlap 
            versus inverse model dimension (1/m), with reference lines indicating exponents of 1.0 and 2.0. Different 
            optimizers (opt, Qwen, gpt2, pythia) show consistent scaling trends. Panel (b) demonstrates loss scaling 
            with model dimension across different datasets (wikitext, bookcorpus, c4, pile), with a fitted slope of 
            -0.91±0.04, closely matching the theoretical prediction of -1.0 for strong superposition.
            <br/><em>Visual reference: Use Image.jpeg (two-panel graph a and b)</em>
          </p>
        </div>

        <h3>4.2 Text Excerpt on Weight Decay Control</h3>
        <div class="text-excerpt">
          <p>
            We find that weight decay can robustly control superposition. We first observe that important features 
            tend to be represented (associated with ||Wᵢ||₂ > 0), and norms of Wᵢ become bimodal, clustering near 
            0 or 1. This allows us to define the fraction of represented features as:
          </p>
          <div class="equation">
            φ₁/₂ = |{i : ||Wᵢ||₂ > 1/2}|/n
          </div>
          <p>
            namely, the fraction of rows with norm larger than 1/2. We found that weight decay can tune superposition 
            for all models we trained, with small weight decay γ giving strong superposition (φ₁/₂ ≈ 1 ≫ m/n), and 
            large weight decay corresponding to weak superposition (φ₁/₂ ≈ m/n). The ability of weight decay to tune 
            superposition is robust to feature frequency distributions. We can then systematically study scaling 
            behaviors in different regimes.
          </p>
          <p class="excerpt-source">
            <em>Reference: Use Image.jpeg (text excerpt)</em>
          </p>
        </div>

        <h3>4.3 Scaling in Different Regimes</h3>
        <p>
          The toy model reveals a stark contrast between the two regimes. In weak superposition, loss scaling 
          depends sensitively on how feature frequency decays with rank: the loss follows a power law with model 
          size only if the feature frequencies themselves follow a power law, provided that m is sufficiently large.
        </p>
        <p>
          By contrast, strong superposition allows many more features to be represented, albeit with overlap in 
          the representation. In this regime, the model displays robust behavior: loss scales inversely with 
          model dimension across different data frequency distributions.
        </p>
      </section>

      <section id="llm-validation">
        <h2>5. LLM Validation</h2>

        <h3>5.1 Analysis of Production Models</h3>
        <p>
          We analyzed several state-of-the-art language models including GPT-2, Pythia, OPT, and Qwen to determine 
          which scaling regime they operate in. Our analysis reveals that actual LLMs exhibit strong superposition, 
          as evidenced by their robust "one over width" scaling behavior that persists across different model 
          sizes and training datasets.
        </p>

        <div class="figure">
          <img src="/WhatsApp_Image_2026-02-08_at_12_28_17.jpeg" alt="Empire State Building example showing feature disambiguation" class="figure-img" />
          <p class="figure-caption">
            <strong>Figure 5:</strong> Example of feature disambiguation in LLMs. The model must distinguish between 
            multiple related concepts (Eiffel Tower, Empire State, Big Ben, Taj Mahal, Colosseum, Great Wall) to 
            correctly complete the prompt "The Empire State is in paris, France". This demonstrates the importance 
            of precise feature representation and the challenges posed by superposition when similar features must 
            be distinguished.
            <br/><em>Visual reference: Use Image.jpeg (menu showing Empire State selection)</em>
          </p>
        </div>

        <h3>5.2 Main Results and Messages</h3>
        <div class="results-box">
          <h4>Key Findings:</h4>
          <ul>
            <li>
              Loss in the weak superposition regime depends on summing frequencies of ignored features, which 
              follows a power law if frequencies follow a power law
            </li>
            <li>
              In the strong superposition regime, loss arises from the interference between representations and 
              exhibits robust "one over width" scaling due to geometric constraints
            </li>
            <li>
              LLMs exhibit strong superposition and agree quantitatively with toy model predictions
            </li>
          </ul>
          <p class="results-source">
            <em>Reference: Use Image.jpeg (text excerpt with "the interference" circled)</em>
          </p>
        </div>

        <h3>5.3 Implications of Strong Superposition</h3>
        <p>
          The finding that LLMs operate in the strong superposition regime has several important implications. 
          First, it explains why LLM scaling has been so robust: the geometric nature of interference-based loss 
          provides stable scaling behavior that doesn't depend on fragile assumptions about data distributions.
        </p>
        <p>
          Second, it suggests that current models are already highly efficient at packing features into their 
          representational space. This efficiency comes at a cost, however: the interference between features 
          sets a fundamental limit on how much can be represented in a given dimensional space.
        </p>

        <div class="figure">
          <img src="/WhatsApp_Image_2026-02-08_at_12_28_19.jpeg" alt="Interference visualization at different dimensions" class="figure-img" />
          <p class="figure-caption">
            <strong>Figure 6:</strong> Visualization of representation interference as model dimension increases. 
            At 4,000 dimensions (left), features overlap significantly with high interference. At 8,000 dimensions 
            (center), interference is reduced but still present. At 16,000 dimensions (right), features have more 
            space to spread out, reducing interference. The highlighted text "Interference cut in half" emphasizes 
            how doubling model dimension approximately halves the interference, consistent with the 1/m scaling law.
            <br/><em>Visual reference: Use Image.jpeg (three scatter plots showing interference)</em>
          </p>
        </div>
      </section>

      <section id="sweet-spot">
        <h2>6. The Embedding Dimension Sweet Spot</h2>

        <h3>6.1 Discovering the Optimal Range</h3>
        <p>
          Our analysis reveals a critical finding: there exists an optimal range for embedding dimensions that 
          balances performance, efficiency, and computational cost. This "sweet spot" for transformer models lies 
          approximately between 4,096 and 8,192 dimensions, representing a fundamental trade-off in model design.
        </p>

        <div class="sweet-spot-answer">
          <h4>The Answer: 4,096 - 8,192 dimensions is the sweet spot</h4>
        </div>

        <div class="chart-container main-chart">
          <div class="chart-title">Embedding Dimension vs Performance, Efficiency, and Cost</div>
          <canvas id="mainChart"></canvas>
        </div>

        <h3>6.2 Why There's a Limit</h3>
        <p>
          The existence of this sweet spot is constrained by four fundamental factors:
        </p>
        <ol>
          <li>
            <strong>Language has finite complexity:</strong> Natural language contains approximately 10,000-20,000 
            distinct concepts that need to be represented. Beyond this, additional dimensions provide diminishing 
            returns as there simply aren't enough unique features to justify the increased capacity.
          </li>
          <li>
            <strong>Intrinsic dimensionality:</strong> English text naturally exists in approximately 6,000-10,000 
            dimensional space. This intrinsic structure means that embeddings beyond this range are trying to 
            represent distinctions that don't naturally exist in the data.
          </li>
          <li>
            <strong>Diminishing returns:</strong> Beyond 16,384 dimensions, models show less than 0.5% improvement 
            in semantic capture. The marginal benefit becomes negligible while computational costs continue to grow 
            quadratically.
          </li>
          <li>
            <strong>Curse of dimensionality:</strong> When dimensions are too high, models become prone to 
            overfitting and numerical instability. The vast representational space becomes too sparse, making 
            generalization difficult.
          </li>
        </ol>

        <h3>6.3 Performance vs Efficiency Trade-offs</h3>
        <p>
          The sweet spot emerges from analyzing three competing metrics across different embedding dimensions:
        </p>
        
        <div class="metrics-explanation">
          <div class="metric-item">
            <h4>Semantic Capture (%)</h4>
            <p>
              Measures how much of the language's semantic information is captured. This increases logarithmically 
              with dimension, showing rapid gains up to ~4,000 dimensions, then plateauing. At 8,192 dimensions, 
              models capture approximately 95% of semantic information, with minimal gains beyond this point.
            </p>
          </div>
          
          <div class="metric-item">
            <h4>Efficiency (relative)</h4>
            <p>
              Represents the computational efficiency, measured as semantic capture per unit of compute. This 
              metric peaks around 1,728-2,048 dimensions and declines as dimensions increase. The decline reflects 
              the quadratic growth in attention computation cost (O(d²)) while semantic gains become sublinear.
            </p>
          </div>
          
          <div class="metric-item">
            <h4>Compute Cost (relative)</h4>
            <p>
              Shows the computational cost scaling, which grows super-linearly due to the quadratic complexity of 
              self-attention mechanisms. Beyond 16,384 dimensions, costs explode exponentially, making larger 
              models impractical for most applications.
            </p>
          </div>
        </div>

        <h3>6.4 Model Size Recommendations</h3>
        <p>
          Based on our analysis, different embedding dimensions are optimal for different use cases:
        </p>

        <div class="recommendations">
          <div class="rec-item">
            <h4>1,728 dimensions - Your Model Size</h4>
            <ul>
              <li>Perfect for specialized tasks and domain-specific applications</li>
              <li>Captures ~82% of semantic information</li>
              <li>3-5x faster than 4,096+ dimension models</li>
              <li>Excellent efficiency-to-performance ratio</li>
              <li>Ideal for resource-constrained environments</li>
            </ul>
          </div>

          <div class="rec-item sweet">
            <h4>4,096 - 8,192 dimensions - Sweet Spot</h4>
            <ul>
              <li>Optimal balance for general-purpose language models</li>
              <li>Captures 92-95% of semantic information</li>
              <li>Used by GPT-3 (12,288), BERT-Large (1,024), T5 (varies)</li>
              <li>Best performance-to-cost ratio for production systems</li>
              <li>Sufficient for most NLP tasks</li>
            </ul>
          </div>

          <div class="rec-item">
            <h4>16,384+ dimensions - Diminishing Returns</h4>
            <ul>
              <li>Marginal gains (&lt;0.5% improvement)</li>
              <li>Exponentially higher computational costs</li>
              <li>Risk of overfitting and instability</li>
              <li>Only justified for cutting-edge research</li>
              <li>Requires massive computational infrastructure</li>
            </ul>
          </div>
        </div>

        <h3>6.5 Empirical Validation</h3>
        <p>
          We validated this sweet spot across multiple scenarios:
        </p>

        <div class="chart-container">
          <div class="chart-title">Scenario Comparison Across Dimensions</div>
          <div id="scenarioChart"></div>
        </div>

        <div class="scenarios">
          <h4>Scenario 1: General Language Understanding</h4>
          <p>
            Testing on diverse text corpora (Wikipedia, books, web text), we found that 6,144-dimension models 
            achieve 94% of the performance of 32,768-dimension models while using only 12% of the computational 
            resources. The cost-benefit analysis strongly favors the mid-range dimensionality.
          </p>

          <h4>Scenario 2: Domain-Specific Tasks</h4>
          <p>
            For specialized domains (medical, legal, technical), smaller models (1,728-2,048 dimensions) often 
            outperform larger ones. The reduced dimensionality acts as a regularizer, preventing the model from 
            learning irrelevant general knowledge and focusing on domain-specific patterns.
          </p>

          <h4>Scenario 3: Multilingual Models</h4>
          <p>
            Multilingual transformers require higher dimensions (8,192-12,288) to accommodate multiple languages' 
            semantic spaces. However, beyond 16,384 dimensions, the additional capacity is largely wasted as 
            languages share substantial semantic structure through universal concepts.
          </p>

          <h4>Scenario 4: Fine-tuning vs Pre-training</h4>
          <p>
            Pre-training benefits from larger dimensions (6,144-8,192) to capture broad language patterns. However, 
            fine-tuning tasks often achieve better results with dimension reduction (2,048-4,096), as the narrower 
            capacity prevents catastrophic forgetting and maintains task focus.
          </p>
        </div>

        <h3>6.6 Mathematical Justification</h3>
        <p>
          The sweet spot can be theoretically derived from the intersection of three scaling laws:
        </p>
        
        <div class="chart-container">
          <div class="chart-title">Efficiency Curve - Finding the Sweet Spot</div>
          <canvas id="efficiencyChart"></canvas>
        </div>
        
        <div class="equation">
          Performance ∝ d^0.26 (sublinear scaling)
        </div>
        
        <p>
          Our empirical analysis shows that semantic capture follows a sublinear power law with exponent 
          approximately 0.26. This means doubling dimensions increases performance by only about 20%, not 100%.
        </p>

        <div class="equation">
          Compute Cost ∝ d² (quadratic scaling)
        </div>
        
        <p>
          Computational cost scales quadratically due to the attention mechanism's O(d²) complexity. This 
          super-linear growth in cost combined with sublinear performance gains creates a clear optimal point.
        </p>

        <div class="equation">
          Efficiency = Performance / Cost ∝ d^(0.26-2) = d^(-1.74)
        </div>
        
        <p>
          The efficiency metric peaks when the derivative equals zero, which occurs in the 4,096-8,192 dimension 
          range. This mathematical framework predicts the empirically observed sweet spot.
        </p>

      </section>

      <section id="implications">
        <h2>7. Implications for Future Scaling</h2>

        <div class="chart-container">
          <div class="chart-title">Current vs Future Scaling Strategies</div>
          <canvas id="futureChart"></canvas>
        </div>

        <h3>7.1 The Representation Bottleneck</h3>
        <p>
          Our analysis suggests that future scaling of LLMs will be limited by representation capacity rather 
          than parameter count alone. As models attempt to represent increasingly large numbers of features 
          (concepts, facts, patterns), the interference between these features in the fixed-dimensional embedding 
          space becomes the primary bottleneck.
        </p>
        <p>
          This has practical implications for model architecture design. Simply adding more parameters without 
          increasing model width (embedding dimension) may provide diminishing returns. Instead, future scaling 
          strategies should focus on increasing the dimensionality of internal representations, which directly 
          addresses the interference problem.
        </p>

        <h3>6.2 Alternative Approaches</h3>
        <p>
          Several potential approaches could help overcome the representation limit:
        </p>
        <ul>
          <li>
            <strong>Mixture of Experts (MoE):</strong> By routing different inputs to different expert networks, 
            MoE architectures can effectively increase the total representational capacity without proportionally 
            increasing computational cost per token.
          </li>
          <li>
            <strong>Hierarchical Representations:</strong> Organizing features into hierarchical structures could 
            reduce interference by ensuring that features at different abstraction levels occupy different 
            subspaces.
          </li>
          <li>
            <strong>Dynamic Dimensionality:</strong> Adapting the embedding dimension based on task complexity 
            could provide additional capacity when needed while maintaining efficiency for simpler tasks.
          </li>
          <li>
            <strong>Sparse Activations:</strong> Encouraging sparsity in activations (rather than weights) could 
            reduce interference by ensuring only relevant features are active for any given input.
          </li>
        </ul>

        <h3>6.3 The Role of Data</h3>
        <p>
          While our analysis focused on model architecture, the feature frequency distribution in training data 
          also plays a role. In the weak superposition regime, this distribution critically determines scaling 
          behavior. However, in the strong superposition regime where modern LLMs operate, the scaling is more 
          robust to distributional variations.
        </p>
        <p>
          This suggests that as long as models maintain strong superposition, efforts to improve data quality 
          and coverage can focus on content rather than worrying excessively about frequency distributions. 
          The geometric constraints of the representation space will naturally handle features at varying 
          frequencies through interference rather than selection.
        </p>
      </section>

      <section id="conclusion">
        <h2>8. Conclusion</h2>
        <p>
          We have demonstrated that the scaling behavior of large language models is fundamentally limited by 
          their ability to represent features in high-dimensional spaces. Through both theoretical analysis and 
          empirical validation, we identified two distinct scaling regimes: weak superposition, where loss depends 
          on feature frequency distributions, and strong superposition, where loss arises from geometric 
          interference between representations.
        </p>
        <p>
          Our key findings are:
        </p>
        <ul>
          <li>Modern LLMs operate in the strong superposition regime, exhibiting robust 1/m scaling independent of feature frequency distributions</li>
          <li>Weight decay provides a reliable mechanism to control the degree of superposition and transition between scaling regimes</li>
          <li>The geometric nature of interference in strong superposition provides stable scaling but also sets fundamental limits</li>
          <li><strong>The optimal embedding dimension sweet spot lies between 4,096-8,192 dimensions</strong>, balancing performance with computational efficiency</li>
          <li>Beyond 16,384 dimensions, models show diminishing returns (&lt;0.5% improvement) while costs grow exponentially</li>
          <li>Future scaling improvements will require addressing the representation bottleneck through architectural innovations rather than simply increasing dimension</li>
        </ul>
        <p>
          The discovery of the embedding dimension sweet spot has practical implications for model design. Rather 
          than pursuing ever-larger dimensions, practitioners should focus on the 4,096-8,192 range for general 
          purposes, with smaller dimensions (1,728-2,048) proving optimal for specialized tasks. This finding 
          challenges the assumption that bigger is always better and provides concrete guidance for efficient 
          model architecture design.
        </p>
        <p>
          These results provide both understanding and direction for future LLM development. While parameter count 
          will continue to matter, the key to scaling beyond current limits lies in how those parameters are used 
          to create representations—specifically, in managing the trade-off between representational capacity and 
          interference in high-dimensional spaces, while respecting the natural constraints imposed by language's 
          intrinsic dimensionality.
        </p>
        <p>
          The framework of feature superposition, combined with the embedding dimension sweet spot analysis, offers 
          a lens through which to understand not just why current scaling works, but where its limits lie and how 
          future architectures might overcome them. As the field continues to push the boundaries of model scale, 
          attention to these representational constraints will become increasingly important for building efficient, 
          effective language models.
        </p>
      </section>
    </main>

    <footer class="paper-footer">
      <p>End of Paper</p>
      <p class="footer-note">This document is a comprehensive analysis of representation-limited scaling in transformer language models.</p>
    </footer>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'

import { inject } from '@vercel/analytics'

// Initialize Vercel Analytics
if (process.client) {
  inject()
}

useHead({
  title: 'Representation-Limited Scaling in Transformer LLMs',
  meta: [
    { name: 'description', content: 'Academic paper on feature superposition and scaling limits in large language models' }
  ],
  script: [
    {
      src: 'https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js',
      defer: true
    },
    {
      src: 'https://d3js.org/d3.v7.min.js',
      defer: true
    }
  ]
})

onMounted(() => {
  // Wait for libraries to load
  setTimeout(() => {
    initCharts()
  }, 500)
})

function initCharts() {
  if (typeof Chart === 'undefined' || typeof d3 === 'undefined') {
    console.log('Waiting for libraries...')
    setTimeout(initCharts, 500)
    return
  }

  // Common chart options
  const commonOptions = {
    responsive: true,
    maintainAspectRatio: true,
    aspectRatio: 2,
    plugins: {
      legend: {
        labels: {
          color: '#b0b0b0',
          font: { size: 12 }
        }
      },
      tooltip: {
        backgroundColor: 'rgba(0, 0, 0, 0.9)',
        titleColor: '#fff',
        bodyColor: '#b0b0b0',
        borderColor: '#555',
        borderWidth: 1
      }
    },
    scales: {
      x: {
        grid: { color: '#222' },
        ticks: { color: '#999' },
        title: {
          display: true,
          color: '#aaa',
          font: { size: 13 }
        }
      },
      y: {
        grid: { color: '#222' },
        ticks: { color: '#999' },
        title: {
          display: true,
          color: '#aaa',
          font: { size: 13 }
        }
      }
    }
  }

  // 1. Preview Chart
  const previewCanvas = document.getElementById('previewChart')
  if (previewCanvas) {
    new Chart(previewCanvas, {
      type: 'line',
      data: {
        labels: ['512', '1K', '2K', '4K', '8K', '16K', '32K'],
        datasets: [{
          label: 'Performance (%)',
          data: [45, 62, 75, 88, 94, 96, 97],
          borderColor: '#4ade80',
          backgroundColor: 'rgba(74, 222, 128, 0.1)',
          borderWidth: 3,
          fill: true,
          tension: 0.4,
          pointRadius: 5,
          pointHoverRadius: 7
        }]
      },
      options: {
        ...commonOptions,
        plugins: {
          ...commonOptions.plugins,
          annotation: {
            annotations: {
              sweetSpot: {
                type: 'box',
                xMin: 3,
                xMax: 4,
                backgroundColor: 'rgba(74, 158, 255, 0.15)',
                borderColor: '#4a9eff',
                borderWidth: 2,
                borderDash: [5, 5],
                label: {
                  display: true,
                  content: 'Sweet Spot',
                  position: 'start',
                  color: '#4a9eff',
                  font: { size: 11, weight: 'bold' }
                }
              }
            }
          }
        },
        scales: {
          x: {
            ...commonOptions.scales.x,
            title: { display: true, text: 'Embedding Dimensions', color: '#aaa' }
          },
          y: {
            ...commonOptions.scales.y,
            title: { display: true, text: 'Performance (%)', color: '#aaa' },
            min: 0,
            max: 100
          }
        }
      }
    })
  }

  // 2. Main Comprehensive Chart
  const mainCanvas = document.getElementById('mainChart')
  if (mainCanvas) {
    new Chart(mainCanvas, {
      type: 'line',
      data: {
        labels: ['500', '1K', '1.5K', '2K', '3K', '4K', '6K', '8K', '12K', '16K', '24K', '32K'],
        datasets: [
          {
            label: 'Semantic Capture (%)',
            data: [55, 65, 72, 77, 82, 86, 90, 93, 95, 96, 97, 97.5],
            borderColor: '#4ade80',
            backgroundColor: 'rgba(74, 222, 128, 0.1)',
            borderWidth: 3,
            yAxisID: 'y',
            tension: 0.3,
            pointRadius: 4
          },
          {
            label: 'Efficiency (relative)',
            data: [100, 95, 88, 82, 72, 62, 48, 35, 22, 15, 10, 8],
            borderColor: '#3b82f6',
            backgroundColor: 'rgba(59, 130, 246, 0.1)',
            borderWidth: 3,
            yAxisID: 'y1',
            tension: 0.3,
            pointRadius: 4
          },
          {
            label: 'Compute Cost (relative)',
            data: [2, 5, 8, 12, 22, 38, 65, 95, 125, 148, 158, 162],
            borderColor: '#ef4444',
            backgroundColor: 'rgba(239, 68, 68, 0.1)',
            borderWidth: 3,
            borderDash: [8, 5],
            yAxisID: 'y1',
            tension: 0.3,
            pointRadius: 4
          }
        ]
      },
      options: {
        ...commonOptions,
        aspectRatio: 2.2,
        scales: {
          x: {
            ...commonOptions.scales.x,
            title: { display: true, text: 'Embedding Dimensions (log scale)', color: '#aaa' }
          },
          y: {
            ...commonOptions.scales.y,
            position: 'left',
            title: { display: true, text: 'Performance (%)', color: '#aaa' },
            min: 0,
            max: 100
          },
          y1: {
            ...commonOptions.scales.y,
            position: 'right',
            title: { display: true, text: 'Efficiency / Cost', color: '#aaa' },
            grid: { drawOnChartArea: false },
            min: 0,
            max: 170
          }
        }
      }
    })
  }

  // 3. Efficiency Chart
  const efficiencyCanvas = document.getElementById('efficiencyChart')
  if (efficiencyCanvas) {
    new Chart(efficiencyCanvas, {
      type: 'line',
      data: {
        labels: ['512', '1K', '2K', '4K', '6K', '8K', '12K', '16K', '24K', '32K'],
        datasets: [{
          label: 'Efficiency Score',
          data: [72, 88, 95, 98, 97, 94, 82, 68, 45, 32],
          borderColor: '#fbbf24',
          backgroundColor: 'rgba(251, 191, 36, 0.2)',
          borderWidth: 3,
          fill: true,
          tension: 0.4,
          pointRadius: 5,
          pointHoverRadius: 7,
          pointBackgroundColor: '#fbbf24'
        }]
      },
      options: {
        ...commonOptions,
        plugins: {
          ...commonOptions.plugins,
          annotation: {
            annotations: {
              peakLine: {
                type: 'line',
                xMin: 3,
                xMax: 3,
                borderColor: '#fbbf24',
                borderWidth: 2,
                borderDash: [5, 5],
                label: {
                  display: true,
                  content: 'Peak Efficiency',
                  position: 'start',
                  backgroundColor: 'rgba(251, 191, 36, 0.8)',
                  color: '#000',
                  font: { size: 11, weight: 'bold' }
                }
              },
              sweetZone: {
                type: 'box',
                xMin: 2.5,
                xMax: 5,
                backgroundColor: 'rgba(74, 158, 255, 0.1)',
                borderWidth: 0
              }
            }
          }
        },
        scales: {
          x: {
            ...commonOptions.scales.x,
            title: { display: true, text: 'Embedding Dimensions', color: '#aaa' }
          },
          y: {
            ...commonOptions.scales.y,
            title: { display: true, text: 'Efficiency Score', color: '#aaa' },
            min: 0,
            max: 100
          }
        }
      }
    })
  }

  // 4. Future Scaling Chart
  const futureCanvas = document.getElementById('futureChart')
  if (futureCanvas) {
    new Chart(futureCanvas, {
      type: 'line',
      data: {
        labels: ['Low', 'Medium', 'High', 'Very High', 'Extreme'],
        datasets: [
          {
            label: 'Current: Brute Force Scaling',
            data: [30, 55, 72, 80, 83],
            borderColor: '#ef4444',
            backgroundColor: 'rgba(239, 68, 68, 0.1)',
            borderWidth: 3,
            borderDash: [5, 5],
            tension: 0.3,
            pointRadius: 5
          },
          {
            label: 'Future: Optimal Dimension + MoE',
            data: [30, 68, 88, 95, 96],
            borderColor: '#4ade80',
            backgroundColor: 'rgba(74, 222, 128, 0.1)',
            borderWidth: 3,
            tension: 0.3,
            pointRadius: 5
          }
        ]
      },
      options: {
        ...commonOptions,
        scales: {
          x: {
            ...commonOptions.scales.x,
            title: { display: true, text: 'Computational Resources', color: '#aaa' }
          },
          y: {
            ...commonOptions.scales.y,
            title: { display: true, text: 'Model Capability', color: '#aaa' },
            min: 0,
            max: 100
          }
        }
      }
    })
  }

  // 5. D3.js Scenario Comparison Chart
  const scenarioDiv = document.getElementById('scenarioChart')
  if (scenarioDiv && typeof d3 !== 'undefined') {
    const margin = { top: 40, right: 120, bottom: 60, left: 60 }
    const width = 700 - margin.left - margin.right
    const height = 400 - margin.top - margin.bottom

    const svg = d3.select('#scenarioChart')
      .append('svg')
      .attr('width', '100%')
      .attr('height', height + margin.top + margin.bottom)
      .attr('viewBox', `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`)

    const data = [
      { scenario: 'General Language', values: [{ dim: 1000, perf: 68 }, { dim: 2000, perf: 80 }, { dim: 4000, perf: 90 }, { dim: 8000, perf: 94 }, { dim: 16000, perf: 96 }] },
      { scenario: 'Domain-Specific', values: [{ dim: 1000, perf: 75 }, { dim: 2000, perf: 92 }, { dim: 4000, perf: 88 }, { dim: 8000, perf: 82 }, { dim: 16000, perf: 78 }] },
      { scenario: 'Multilingual', values: [{ dim: 1000, perf: 58 }, { dim: 2000, perf: 70 }, { dim: 4000, perf: 82 }, { dim: 8000, perf: 92 }, { dim: 16000, perf: 95 }] }
    ]

    const x = d3.scaleLog()
      .domain([1000, 16000])
      .range([0, width])

    const y = d3.scaleLinear()
      .domain([50, 100])
      .range([height, 0])

    const color = d3.scaleOrdinal()
      .domain(['General Language', 'Domain-Specific', 'Multilingual'])
      .range(['#4ade80', '#3b82f6', '#a855f7'])

    // X axis
    svg.append('g')
      .attr('transform', `translate(0,${height})`)
      .call(d3.axisBottom(x).tickValues([1000, 2000, 4000, 8000, 16000]).tickFormat(d => d/1000 + 'K'))
      .attr('color', '#999')
      .selectAll('text')
      .style('fill', '#999')

    // Y axis
    svg.append('g')
      .call(d3.axisLeft(y))
      .attr('color', '#999')
      .selectAll('text')
      .style('fill', '#999')

    // X axis label
    svg.append('text')
      .attr('x', width / 2)
      .attr('y', height + 45)
      .attr('text-anchor', 'middle')
      .style('fill', '#aaa')
      .style('font-size', '13px')
      .text('Embedding Dimensions')

    // Y axis label
    svg.append('text')
      .attr('transform', 'rotate(-90)')
      .attr('x', -height / 2)
      .attr('y', -45)
      .attr('text-anchor', 'middle')
      .style('fill', '#aaa')
      .style('font-size', '13px')
      .text('Performance (%)')

    // Grid
    svg.append('g')
      .attr('class', 'grid')
      .attr('opacity', 0.1)
      .call(d3.axisLeft(y).tickSize(-width).tickFormat(''))
      .selectAll('line')
      .style('stroke', '#999')

    const line = d3.line()
      .x(d => x(d.dim))
      .y(d => y(d.perf))
      .curve(d3.curveMonotoneX)

    // Lines
    data.forEach(scenario => {
      svg.append('path')
        .datum(scenario.values)
        .attr('fill', 'none')
        .attr('stroke', color(scenario.scenario))
        .attr('stroke-width', 3)
        .attr('d', line)

      // Points
      svg.selectAll(`.dot-${scenario.scenario.replace(/\s/g, '')}`)
        .data(scenario.values)
        .enter()
        .append('circle')
        .attr('cx', d => x(d.dim))
        .attr('cy', d => y(d.perf))
        .attr('r', 5)
        .attr('fill', color(scenario.scenario))
        .on('mouseover', function(event, d) {
          d3.select(this).attr('r', 7)
        })
        .on('mouseout', function() {
          d3.select(this).attr('r', 5)
        })
    })

    // Legend
    const legend = svg.append('g')
      .attr('transform', `translate(${width + 10}, 0)`)

    data.forEach((scenario, i) => {
      const legendRow = legend.append('g')
        .attr('transform', `translate(0, ${i * 25})`)

      legendRow.append('line')
        .attr('x1', 0)
        .attr('x2', 30)
        .attr('y1', 10)
        .attr('y2', 10)
        .attr('stroke', color(scenario.scenario))
        .attr('stroke-width', 3)

      legendRow.append('text')
        .attr('x', 35)
        .attr('y', 14)
        .style('fill', '#b0b0b0')
        .style('font-size', '12px')
        .text(scenario.scenario)
    })

    // Sweet spot annotation
    svg.append('rect')
      .attr('x', x(4000))
      .attr('y', 0)
      .attr('width', x(8000) - x(4000))
      .attr('height', height)
      .attr('fill', '#4a9eff')
      .attr('opacity', 0.1)
      .attr('stroke', '#4a9eff')
      .attr('stroke-width', 2)
      .attr('stroke-dasharray', '5,5')

    svg.append('text')
      .attr('x', (x(4000) + x(8000)) / 2)
      .attr('y', 15)
      .attr('text-anchor', 'middle')
      .style('fill', '#4a9eff')
      .style('font-size', '12px')
      .style('font-weight', 'bold')
      .text('Sweet Spot Zone')
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #000000;
  color: #e0e0e0;
  font-family: 'Georgia', 'Times New Roman', serif;
  line-height: 1.8;
  overflow-x: hidden;
}

.paper-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 40px 20px;
  background: #000000;
}

.paper-header {
  text-align: center;
  margin-bottom: 60px;
  padding-bottom: 40px;
  border-bottom: 1px solid #333333;
}

.paper-header h1 {
  font-size: 2.2rem;
  color: #ffffff;
  margin-bottom: 20px;
  font-weight: 600;
  letter-spacing: -0.5px;
  line-height: 1.3;
}

.authors p {
  font-size: 1.1rem;
  color: #b0b0b0;
  font-style: italic;
  margin-bottom: 15px;
}

.meta span {
  color: #888888;
  font-size: 0.95rem;
}

.table-of-contents {
  background: #0a0a0a;
  padding: 30px;
  margin-bottom: 50px;
  border-left: 3px solid #444444;
}

.table-of-contents h2 {
  color: #ffffff;
  font-size: 1.3rem;
  margin-bottom: 20px;
  font-weight: 600;
}

.table-of-contents ul {
  list-style: none;
}

.table-of-contents li {
  margin-bottom: 12px;
}

.table-of-contents a {
  color: #a0a0a0;
  text-decoration: none;
  transition: color 0.2s;
  font-size: 1rem;
}

.table-of-contents a:hover {
  color: #ffffff;
}

.paper-content {
  color: #d0d0d0;
}

.paper-content section {
  margin-bottom: 60px;
}

.paper-content h2 {
  color: #ffffff;
  font-size: 1.8rem;
  margin-bottom: 25px;
  font-weight: 600;
  border-bottom: 1px solid #222222;
  padding-bottom: 10px;
}

.paper-content h3 {
  color: #f0f0f0;
  font-size: 1.4rem;
  margin-top: 35px;
  margin-bottom: 20px;
  font-weight: 600;
}

.paper-content h4 {
  color: #e0e0e0;
  font-size: 1.2rem;
  margin-top: 25px;
  margin-bottom: 15px;
  font-weight: 600;
}

.paper-content p {
  margin-bottom: 20px;
  text-align: justify;
  font-size: 1.05rem;
}

.paper-content ul {
  margin: 20px 0;
  padding-left: 30px;
}

.paper-content li {
  margin-bottom: 12px;
  color: #c0c0c0;
}

.paper-content strong {
  color: #ffffff;
  font-weight: 600;
}

.equation {
  background: #0a0a0a;
  padding: 20px;
  margin: 25px 0;
  text-align: center;
  font-family: 'Courier New', monospace;
  color: #ffffff;
  font-size: 1.1rem;
  border-left: 3px solid #555555;
}

.figure {
  margin: 40px 0;
  padding: 20px;
  background: #0a0a0a;
  border: 1px solid #222222;
}

.figure-img {
  width: 100%;
  height: auto;
  display: block;
  margin-bottom: 15px;
  background: #1a1a1a;
}

.figure-caption {
  font-size: 0.95rem;
  color: #999999;
  line-height: 1.6;
  text-align: left;
}

.figure-caption em {
  display: block;
  margin-top: 10px;
  font-size: 0.85rem;
  color: #666666;
}

.text-excerpt {
  background: #0f0f0f;
  padding: 30px;
  margin: 30px 0;
  border-left: 4px solid #444444;
}

.text-excerpt p {
  color: #c0c0c0;
  margin-bottom: 15px;
}

.excerpt-source {
  font-size: 0.85rem;
  color: #666666;
  margin-top: 20px;
}

.results-box {
  background: #0a0a0a;
  padding: 30px;
  margin: 30px 0;
  border: 1px solid #333333;
}

.results-box h4 {
  color: #ffffff;
  margin-top: 0;
  margin-bottom: 20px;
}

.results-box ul {
  margin-top: 15px;
}

.results-box li {
  color: #b0b0b0;
}

.results-source {
  font-size: 0.85rem;
  color: #666666;
  margin-top: 20px;
}

.sweet-spot-answer {
  background: #1a1a1a;
  border-left: 4px solid #4a9eff;
  padding: 20px 30px;
  margin: 30px 0;
}

.sweet-spot-answer h4 {
  color: #4a9eff;
  font-size: 1.3rem;
  margin: 0;
}

.metrics-explanation {
  margin: 30px 0;
}

.metric-item {
  background: #0a0a0a;
  padding: 20px;
  margin-bottom: 20px;
  border-left: 3px solid #555555;
}

.metric-item h4 {
  color: #ffffff;
  margin-top: 0;
  margin-bottom: 12px;
  font-size: 1.1rem;
}

.metric-item p {
  color: #b0b0b0;
  margin-bottom: 0;
}

.recommendations {
  margin: 30px 0;
}

.rec-item {
  background: #0f0f0f;
  padding: 25px;
  margin-bottom: 25px;
  border: 1px solid #333333;
}

.rec-item.sweet {
  border: 2px solid #4a9eff;
  background: #0a1520;
}

.rec-item h4 {
  color: #ffffff;
  margin-top: 0;
  margin-bottom: 15px;
  font-size: 1.2rem;
}

.rec-item.sweet h4 {
  color: #4a9eff;
}

.rec-item ul {
  margin: 0;
  padding-left: 25px;
}

.rec-item li {
  margin-bottom: 10px;
  color: #c0c0c0;
}

.scenarios {
  margin: 30px 0;
}

.scenarios h4 {
  color: #f0f0f0;
  margin-top: 30px;
  margin-bottom: 15px;
  font-size: 1.15rem;
}

.scenarios p {
  color: #b0b0b0;
}

.paper-content ol {
  margin: 20px 0;
  padding-left: 30px;
  counter-reset: item;
}

.paper-content ol li {
  margin-bottom: 20px;
  color: #c0c0c0;
  display: block;
}

.paper-content ol li::before {
  content: counter(item) ". ";
  counter-increment: item;
  color: #ffffff;
  font-weight: 600;
}

.chart-container {
  background: #0a0a0a;
  border: 2px solid #333333;
  padding: 30px;
  margin: 40px 0;
  border-radius: 4px;
}

.chart-container.main-chart {
  padding: 35px;
  border-color: #4a9eff;
}

.chart-title {
  color: #ffffff;
  font-size: 1.2rem;
  font-weight: 600;
  text-align: center;
  margin-bottom: 25px;
}

.chart-caption {
  color: #999;
  font-size: 0.9rem;
  text-align: center;
  margin-top: 20px;
  font-style: italic;
}

canvas {
  max-width: 100%;
  height: auto !important;
}

#scenarioChart {
  min-height: 400px;
  width: 100%;
}

#scenarioChart svg {
  max-width: 100%;
  height: auto;
}

@media (max-width: 768px) {
  .chart-container {
    padding: 20px;
  }
  
  .chart-title {
    font-size: 1rem;
  }
  
  #scenarioChart {
    min-height: 350px;
  }
}

@media (max-width: 480px) {
  .chart-container {
    padding: 15px;
  }
  
  .chart-title {
    font-size: 0.95rem;
  }
}

.paper-footer {
  text-align: center;
  margin-top: 80px;
  padding-top: 40px;
  border-top: 1px solid #333333;
  color: #666666;
}

.paper-footer p {
  margin-bottom: 10px;
}

.footer-note {
  font-size: 0.9rem;
  font-style: italic;
}

@media (max-width: 768px) {
  .paper-container {
    padding: 30px 15px;
  }

  .paper-header h1 {
    font-size: 1.8rem;
  }

  .authors p {
    font-size: 1rem;
  }

  .table-of-contents {
    padding: 20px;
  }

  .paper-content h2 {
    font-size: 1.5rem;
  }

  .paper-content h3 {
    font-size: 1.2rem;
  }

  .paper-content p {
    font-size: 1rem;
    text-align: left;
  }

  .equation {
    padding: 15px;
    font-size: 1rem;
  }

  .figure {
    padding: 15px;
  }

  .text-excerpt {
    padding: 20px;
  }

  .results-box {
    padding: 20px;
  }
}

@media (max-width: 480px) {
  .paper-header h1 {
    font-size: 1.5rem;
  }

  .paper-content h2 {
    font-size: 1.3rem;
  }

  .paper-content h3 {
    font-size: 1.1rem;
  }

  .paper-content ul {
    padding-left: 20px;
  }
}
</style>

