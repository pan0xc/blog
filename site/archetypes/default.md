+++
date = '{{ .Date }}'
draft = true
title = '{{ replaceRE "^[0-9]{4}-[0-9]{2}-[0-9]{2}-" "" .File.ContentBaseName | replaceRE "-" " " | title }}'
authors = [
  "Chen Pan",
]
+++
