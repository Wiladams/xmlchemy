# xmlchemy
Components for dealing with XML.

Some important key points

-- Deals with in-memory buffer
-- No file system or other streaming apis
-- Does no allocations by default
-- Structured for composition


The lowest level thing you can do is use the tokenizer.  Like this:

//=========================================
#include "xmltoken.h"

static void printToken(const XmlToken& tok)
{
	ByteSpan kindname{};
	getEnumKey(XML_TOKEN_TYPE_WSEnum, tok.type, kindname);
	printf("Token [%d] %.*s : [%.*s]\n",
		tok.type, (int)kindname.size(), kindname.data(),
		(int)tok.value.size(), tok.value.data());
}


const char * xmlText = "<svg><rect width='100' height='150' /></svg>";
ByteSpan xmlTextSpan(xmlText);
XmlTokenState fState{xmlTextSpan, false};

XmlToken tok{};
while (nextXmlToken(fState, tok))
{
    printToken(tok);
}
//=========================================

Will generate the stream of tokens

Token [1] < : []
Token [7] NAME : [svg]
Token [2] > : []
Token [1] < : []
Token [7] NAME : [rect]
Token [7] NAME : [width]
Token [4] = : []
Token [8] STRING : [100]
Token [7] NAME : [height]
Token [4] = : []
Token [8] STRING : [150]
Token [3] / : []
Token [2] > : []
Token [1] < : []
Token [3] / : []
Token [7] NAME : [svg]
Token [2] > : []

That lone is enough to do something transcoding, like from XML to JSON.

The next level of composition is the XmlPull, which is setup similarly:

//======================================
#include "xmlscan.h"

XmlPull puller();
XmlElement elem{};

while (puller.next(elem))
{
    printXmlElement(elem);
}

//======================================

The xml scanner will return element beginnings, endings, content, and attributes as a solid span (not parsed).  The initial scan does not do entity expansions, or whitespace collapse.  These steps can be applied if and when the consumer chooses to use them.  

Not splitting out the attributes early allows the scanner to remain "allocation free".  You will get a ByteSpan (fData), which contains the span of memory where the attributes are found.  That saves you from having a std::vector or other data structure carried by the XmlElement.  There is a 'getRawAttributeValue()' function which you can call, and get the attribute key/value out, still without allocations.  You can choose to call that, or other attribute splitting functions if you need, and put the values into a data structure of your choosing.

