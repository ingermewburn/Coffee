---
Item Type: {{itemType}}
Title: "{{title}}"
Authors: {{authors}}
Date: {{date}}
Date Added: {{dateAdded}}
URL: {{url}}
DOI: {{DOI}}
Cite key: {{citekey}}
Topics: {{collections}}
tags: {{allTags}}
Zotero Links: {{uri}}
aliases: 
publisher: "{{publicationTitle}}"
Related:: {% for relation in relations -%} {%- if relation.title -%} [[{{relation.title}}]], {% endif -%} {%- endfor%}
---

{% if isFirstImport %}

# {{title}}

> [!ABSTRACT]- ABSTRACT
> {% if abstractNote %} 
> {{abstractNote|replace("\n"," ")}}
> {% else %} There is no abstract.
> {% endif %}


## Main ideas:
- 
## Methodology:
- 
## Results:
- 
## Key points:
- 
{% endif %}
## Reading notes
{%-
    set zoteroColors = {
        "#ff6666": "red",
        "#f19837": "orange",
        "#5fb236": "green",
        "#ffd400": "yellow",
        "#2ea8e5": "blue",
        "#aaaaaa": "grey"
    }
-%}

{%-
   set colorHeading = {
		"red": "🟥 Very important or Critical",
		"orange": "⚠️ Disagree with author",
		"green": "✅ Supporting Argument or Example",
		"yellow": "⭐ Interesting point",
	      "blue": "📃 Interesting references",
	      "grey": "📅 Vocabulary, Names, Dates, Definitions"
   }
-%}

{%- macro calloutHeader(type) -%}
    {%- switch type -%}
        {%- case "highlight" -%}
        Highlight
        {%- case "image" -%}
        Image
        {%- default -%}
        Note
    {%- endswitch -%}
{%- endmacro %}

{%- set newAnnot = [] -%}
{%- set newAnnotations = [] -%}
{%- set annotations = annotations | filterby("date", "dateafter", lastImportDate) %}

{% if annotations.length > 0 %}
*Imported: {{importDate | format("YYYY-MM-DD HH:mm")}}*

{%- for annot in annotations -%}

    {%- if annot.color in zoteroColors -%}
        {%- set customColor = zoteroColors[annot.color] -%}
    {%- elif annot.colorCategory|lower in colorHeading -%}
    	{%- set customColor = annot.colorCategory|lower -%}
    {%- else -%}
	    {%- set customColor = "other" -%}
    {%- endif -%}

    {%- set newAnnotations = (newAnnotations.push({"annotation": annot, "customColor": customColor}), newAnnotations) -%}

{%- endfor -%}

{#- INSERT ANNOTATIONS -#}
{#- Loops through each of the available colors and only inserts matching annotations -#}
{#- This is a workaround for inserting categories in a predefined order (instead of using groupby & the order in which they appear in the PDF) -#}

{%- for color, heading in colorHeading -%}
{%- for entry in newAnnotations | filterby ("customColor", "startswith", color) -%}
{%- set annot = entry.annotation -%}

{%- if entry and loop.first %}

### {{colorHeading[color]}}
{%- endif %}

> [!quote{{"|" + color if color != "other"}}]+ {{calloutHeader(annot.type)}} ([page. {{annot.pageLabel}}](zotero://open-pdf/library/items/{{annot.attachment.itemKey}}?page={{annot.pageLabel}}&annotation={{annot.id}}))

{%- if annot.annotatedText %}
> {{annot.annotatedText|nl2br}} {% if annot.hashTags %}{{annot.hashTags}}{% endif -%}
{%- endif %}

{%- if annot.imageRelativePath %}
> ![[{{annot.imageRelativePath}}]]
{%- endif %}

{%- if annot.ocrText %}
>
>COMMENT:
> {{annot.ocrText}}
{%- endif %}

{%- if annot.comment %}
>
>COMMENT:
> - {{annot.comment|nl2br}}
{%- endif -%}

{%- endfor -%}
{%- endfor -%}
{% endif %}
