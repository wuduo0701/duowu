# duowu
Hexo + Next 搭建个人博客✏

```
echo "post-receive hook is running..."

GIT_REPO=/home/git/blog.duowu.git
TMP_GIT_CLONE=/tmp/blog.yearito
PUBLIC_WWW=/var/www/blog.yearito

rm -rf $TMP_GIT_CLONE
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}/
```
