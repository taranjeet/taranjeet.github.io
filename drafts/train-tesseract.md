## Tesseract training steps

```
# input file naming convention
[language name].[font name].exp[number].[file extension]

# START from here
# for each training file, do this
tesseract lang.fontname.exp[i].tiff lang.fontname.exp1 batch.nochop makebox

# manually edit each box and save. use some box editor for this

# for each training file, do this
tesseract lang.fontname.exp[i].tiff lang.fontname.exp1 box.train

# for each training file, do this
unicharset_extractor lang.fontname.exp[i].box

# syntax, fontname italic bold monospace serif fraktur
echo "lang 0 0 1 0 0"> font_properties

shapeclustering -F font_properties lang.fontname.exp0.tr lang.fontname.exp1.tr ...each file

mftraining -F font_properties lang.fontname.*.tr

cntraining lang.fontname.*.tr

mv font_properties lang.font_properties;
mv inttemp lang.inttemp;
mv normproto lang.normproto;
mv pffmtable lang.pffmtable;
mv shapetable lang.shapetable;
mv unicharset lang.unicharset;

combine_tessdata lang.
```
