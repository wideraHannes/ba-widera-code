# Explain SHAP for NLP

## Simple Example

To explain how the text "I like big pizzas" is processed by the SHAP explainer, let's assume we have already initialized the TfidfVectorizer, make_predictions function, and the shap.maskers.Text tokenizer as mentioned before.

1. First, the text "I like big pizzas" is passed as input to the SHAP explainer: shap_values = explainer(["I like big pizzas"]).

2. The shap.maskers.Text tokenizer, which uses the regular expression pattern r"\W+", tokenizes the input text. In this case, the tokenizer splits the text into tokens based on non-word characters, resulting in the following tokens: ["I", "like", "big", "pizzas"] coderzcolumn.com coderzcolumn.com.
3. The tokenizer also creates a dictionary with two keys:
   'input_ids': A list of tokens that are not words, such as punctuations or spaces between words. In this case, since there are no non-word tokens, the value of this key would be an empty list [].
   'offset_mapping': A list of tuples representing the start and end indexes of the tokens. In this case, the value of this key would be [(0, 1), (2, 6), (7, 10), (11, 17)], indicating the start and end indexes of each token in the input text coderzcolumn.com coderzcolumn.com.

4. The masked versions of the input text are generated by replacing each token with a mask token. For example, the masked versions of the input text could be: ["[MASK] like big pizzas", "I [MASK] big pizzas", "I like [MASK] pizzas", "I like big [MASK]"] coderzcolumn.com shap.readthedocs.io.
5. The make_predictions function is then called with each masked version of the input text to obtain the model's predictions for each masked instance.
6. The SHAP values are computed based on the differences in the model's predictions between the original text and the masked versions. These SHAP values quantify the contribution of each token to the model's output.

### Nochmal erklärt

So the Tokenizer only defines the granualarity in which we remove players / text.

most of the cases it makes sense to define a player as a word.
so its "[MASK] like big pizzas" and so on passed to the predict function in this case to the pipeline and the text is then vectorized by our defined TF IDF vectorizer

if we want to see the impact on a greater scale we can define a player as a sentence (split by "."). so if the initial sentence is "I like pizzas. But i hate Movies"
then the input to the tokenizer is "[MASK] But i hate Movies" and "I like pizzas [MASK]".

With this we have the flexibility to define what an player is and how big the impact of a player is on the result.

### But what is the Explainer

one missasumption would be to think that The Resulting Shap values are about the applied vectorizer. So the preprocessing steps (in TF-IDF) are applied like lemmatization etc.
The resulting Explanation is at the granularity of the defined Player. So its the Raw text that was initially passed to the Explainer. Without the preprocessing of the Vectorizer. With a sligtly different example:

`Assume we use a Pipeline`

#### Step 1.

`Read the Table from left to right`

| Raw text | Masked Text | Preprocessed (lemmatized, lowercase) | Vector output | Model Prediction |
| -------- | ----------- | ------------------------------------ | ------------- | ---------------- |
| I        | I           | i                                    | [..           | x                |
| liked    | liked       | like                                 | ..            | [*0.1,0.9*]      |
| big      | [masked]    | ''                                   | ..            | x                |
| pizza    | pizza       | pizza                                | ]             | x                |

This is done for every Coalition in the raw text defined by the text Masker.

#### Step 2.

Map the Shap Values / Owen Values to the Raw input text to See What impact each input feature has on the final result

| Raw text | Shap/Owen Value |
| -------- | --------------- |
| I        | 0.1             |
| liked    | 0.4             |
| big      | 0.2             |
| pizza    | 0.2             |

#### Fazit

This is why the Output Text is not Lemmatized or to lowercase etc.

But it is important to remove unnacassary noise like html tags.

The features of the shap_explanation are all words in the text. Split by defined Explainer.
