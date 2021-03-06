# 为 Zotero 删除 storage 中的“空”文件夹

在 Zotero 中使用 Zotfile 时，常会生成一些空文件夹留在 `<数据存储位置>/storage` 中，虽然不占空间，但依然有用户想要删除它们。

> 这些“空”文件夹通常并不是真的没有文件。附件进入 storage 后，Zotero 缓存全文生成 `.zotero-ft-cache`，而后附件被 Zotfile 移动至其他位置，留下只包含 `.zotero-ft-cache` （Linux，MacOS 下为隐藏文件）

以下系统测试通过:

  - Windows10
  - MacOS 10.13.6
  - Manjaro 17.1.11

使用的 Python 均由 [miniconda](https://conda.io/miniconda.html) 安装，版本 3.6.6，**Python 2.x 无法使用**。
建议安装 [click](http://click.pocoo.org/5/)，否则**删除文件夹前不进行确认**。

> 笔者强烈建议任何系统装 Python 直接上 minicoda。

```python
#!/usr/bin/env python
# coding: utf-8

from __future__ import print_function
import sys
import re
import configparser
from pathlib import Path
import shutil

profile_dirs = {
    'darwin': Path.home() / 'Library/Application Support/Zotero',
    'linux': Path.home() / '.zotero/zotero',
    'linux2': Path.home() / '.zotero/zotero',
    'win32': Path.home() / 'AppData/Roaming/Zotero/Zotero'
}
profile_dir = profile_dirs[sys.platform]

config = configparser.ConfigParser()
config.read(profile_dir / 'profiles.ini')
configs_loc = profile_dir / config['Profile0']['Path'] / 'prefs.js'
configs = configs_loc.read_text()

zotero_data_pat = re.compile(r'user_pref\("extensions.zotero.dataDir",\ "(?P<zotero_data>.+)"\);')
zotero_data_dir = Path(zotero_data_pat.search(configs).group('zotero_data'))
storage_dir = zotero_data_dir / 'storage'
dirs_to_remove = [
    p for p in storage_dir.iterdir()
    if (not p.is_file()) and (not len([f for f in list(p.iterdir()) if f.name[0] != '.']))
]

try:
    import click
    print('\n'.join(['The following folders contain no attachments:', *['  {}'.format(p) for p in dirs_to_remove]]))
    if click.confirm('Do you want remove them?', default=True):
        [shutil.rmtree(p, ignore_errors=True) for p in dirs_to_remove]
except ImportError:
    print('\n'.join([
        'The following folders containing no attachments will be removed:', *['  {}'.format(p) for p in dirs_to_remove]
    ]))
    [shutil.rmtree(p, ignore_errors=True) for p in dirs_to_remove]

```

以上脚本也发布在 [GithubGist - zot_rm_empty_folders.py](https://gist.github.com/specter119/0ec043c03d0d8cbe02e83842ee7b2766)，如有bug或者功能上的补充，欢迎在gist上pr。同时欢迎文末留言，但如安装python和有了python如何安装click这类问题，还望自行搜索引擎。
