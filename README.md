# colpali-vision-rag-demo

Query visually rich PDFs without chunking, OCR or messy text extraction. 

Just treat the page as an image and let the model see it.
What this does
Instead of splitting a PDF into chunks like traditional RAG, this demo uses ColPali to embed each entire PDF page as an image. When you ask a question, it finds the most relevant page visually and passes that page image directly to a VLM to answer from it.
Tech Stack
- Byaldi (https://github.com/AnswerDotAI/byaldi) - Minimal wrapper to use ColPali/ColQwen easily
- ColQwen2.5 (https://huggingface.co/vidore/colqwen2.5-v0.2) - The vision retrieval model that does the page-level search
- Qwen2.5-VL (https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct) - The VLM used to answer from the retrieved page image
How to add this to your project
1) Install dependencies
pip install byaldi transformers qwen-vl-utils pillow torch
2) Load the ColPali model
from byaldi import RAGMultiModalModel

model = RAGMultiModalModel.from_pretrained("vidore/colqwen2.5-v0.2", device="cuda")
3) Index your PDF
# This will convert each page to an image and create multi-vector embeddings
model.index(input_path="your_document.pdf", index_name="pdf_index", store_collection_with_index=True)
4) Search for the relevant page
query = "What is the total amount mentioned in this invoice?"
results = model.search(query, k=1)

# results will give you the exact page image to pass to your VLM
retrieved_image = results[0].base64  # or results[0].image
5) Get answer using a VLM
Take that retrieved page image + your query and pass both to Qwen2.5-VL. It'll read the table, text, layout and give you the correct answer directly from that page.
