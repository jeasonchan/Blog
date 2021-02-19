# 0 背景

参考文档：

见pinbox


# 9999
模块：单个py文件

包：某个文件夹且有__init__.py文件，文件下的各个模块时包内的各个模块

比如，python自带json包，这个包下面有很多模块比如decoder.py

import json  导入了一个包，使用decoder还是要  json.decoder  等效于 CPP 里的仅include

from json import decoder    从json包导入了decoder模块，并将decoder符号导入到当前空间，使用时直接使用  decoder  ，等效于CPP里的include + using namespace name;