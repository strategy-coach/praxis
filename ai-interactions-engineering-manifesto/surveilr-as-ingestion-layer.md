
## surveilr document ingestion

Initialize an [RSSD](https://www.surveilr.com/docs/core/concepts/resource-surveillance/) with the initialization command,

```script
surveilr admin init 
```

surveilr file ingestion is explained [here](https://www.surveilr.com/docs/core/cli/cli-commands/#surveilr-ingest-files)

You can use the `surveilr ingest files` command to ingest documents into the system. This command processes the file, extracts relevant content, and converts it into a structured format that can be queried using SQL.

```script
cd www.opsfolio.com/src/ai-context-engineering
surveilr ingest files -r www.opsfolio.com/src/ai-context-engineering/regime/soc2.prompt.md
```

The following command will parse the Markdown file, convert it into JSON, and store it in the `surveilr` database in the uniform_resource_transform table in standardized format.

```script
surveilr orchestrate transform-markdown
```

##### **Querying the Ingested Content Using SQL**

Once the file is ingested into `surveilr`, you can query its contents using  **SQL** . For example, you can query the **document metadata** stored in uniform_resource_transform.

Examples for querying the ingested content.

```sql
-- This set of queries provides extensive examples for testing access to the JSON
-- structure found in the 'content' column of the 'uniform_resource_transform' table.
-- The JSON is an array containing a single object, so all paths start with '$[0]'.

-- The following queries use json_extract() to retrieve specific values.
-- ----------------------------------------------------------------------------------

-- Query 1: Extract the main document title.
-- This query accesses the title at the top level of the document structure.
SELECT
  json_extract(CAST(content AS TEXT), '$[0].items[0].document[0].section.title') AS document_title
FROM uniform_resource_transform
WHERE json_valid(CAST(content AS TEXT));

-- Query 2: Extract a specific subheading title.
-- This query extracts the title of the first second-level section, "What is Opsfolio's SOC 2...".
SELECT
  json_extract(CAST(content AS TEXT), '$[0].items[0].document[0].section.body[0].section.title') AS first_subheading
FROM uniform_resource_transform
WHERE json_valid(CAST(content AS TEXT));

-- Query 3: Extract the paragraph text from a specific section.
-- This query gets the paragraph text under the "Why SOC 2 Matters for Business Growth" heading.
-- We must navigate through the nested arrays and objects to get to the text.
SELECT
  json_extract(CAST(content AS TEXT), '$[0].items[0].document[0].section.body[1].section.body[0].paragraph') AS enterprise_sales_paragraph
FROM uniform_resource_transform
WHERE json_valid(CAST(content AS TEXT));

-- Query 4: Extract a value from a nested list item.
-- This query gets the paragraph text of the first list item under the "Business Outcomes and ROI" section.
-- The path navigates into the 'body' array, then to the 'list' object, then to the 'item' array.
SELECT
  json_extract(CAST(content AS TEXT), '$[0].items[0].document[0].section.body[6].section.body[1].list[0].item[0].paragraph') AS first_list_item_text
FROM uniform_resource_transform
WHERE json_valid(CAST(content AS TEXT));

-- The following queries use json_tree() and json_each() to iterate over the JSON structure.
-- These are useful for dynamic queries or when you don't know the exact path.
-- ----------------------------------------------------------------------------------

-- Query 5: Find all paragraph texts and their paths using json_tree().
-- This is useful for debugging and exploring the entire JSON hierarchy.
SELECT
  json_tree.fullkey AS json_path,
  json_tree.value AS paragraph_text
FROM uniform_resource_transform,
     json_tree(CAST(content AS TEXT)) AS json_tree
WHERE json_tree.key = 'paragraph';

-- Query 6: Find all list items using json_each().
-- This query iterates over the 'list' array to find all of the bullet points
-- from the "Business Outcomes and ROI" section.
SELECT
  json_extract(json_each.value, '$.item[0].paragraph') AS business_outcome
FROM uniform_resource_transform,
     json_each(CAST(content AS TEXT), '$[0].items[0].document[0].section.body[6].section.body[1].list') AS json_each
WHERE json_valid(CAST(content AS TEXT));

-- The following queries demonstrate filtering and more complex joins.
-- ----------------------------------------------------------------------------------

-- Query 7: Find URIs for documents with a specific subheading.
-- This query uses json_tree to find documents where a section title contains a specific phrase.
SELECT
  T1.uri
FROM uniform_resource_transform AS T1,
     json_tree(CAST(T1.content AS TEXT), '$[0].items[0].document[0].section.body') AS T2
WHERE
  T2.key = 'section'
  AND json_extract(T2.value, '$.title') LIKE '%Proven SOC 2 Process%';
```

### surveilr capturable-executables for custom markdown conversion.

[Capturable Executables](https://www.surveilr.com/docs/core/cli/ingest-commands/capexec/) (CEs) are an important feature of `surveilr` that allows users to create custom scripts or processes to extract, transform, and store specific data from documents or other resources. CEs provide a flexible and powerful way to capture relevant content from files, parse it, and insert it into a structured, SQL-queryable format, maintaining the provenance of the data.

### Key Features of Capturable Executables (CEs):

* **Custom Scripts** : CEs allow you to write scripts that process specific documents, extracting only the parts of the data that are relevant (such as headings, tables, or metadata).
* **Flexible Data Capture** : You can use CEs to selectively capture portions of content from complex documents and store them in `surveilr`’s structured format.
* **Provenance Preservation** : CEs track the **lineage** of the data, so you know where the data came from, who processed it, and how it was transformed.

### How to Use Capturable Executables (CEs):

1. **Write the CE Script** :

* You write a custom script that specifies what parts of the document or data to capture. For example, a CE could first extranct documents using [markitdown](https://github.com/microsoft/markitdown). Extract headings, paragraphs, or tables from a Markdown file using custom scripts, then insert that data into the `surveilr` database RSSD with SQL Insert statements.
* CEs can use libraries like Python markitdown, Deno command line invocations or other custom tools to process the content.

1. **Run the CE** :

* Once the script is ready, you can run it through the `surveilr` CLI. The CE will process the file, extract the specified data, and store it in the  `surveilr` SQL database.

2. **Store and Query Data** :

* The extracted data is stored in a **uniform schema** that can be queried using  **SQL** . You can then use SQL queries to retrieve and analyze the ingested content.