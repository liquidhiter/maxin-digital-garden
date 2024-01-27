---
{"dg-publish":true,"permalink":"/0-notes/bash-practice/","noteIcon":"","created":"2024-01-27T07:58:35.679+01:00","updated":"2024-01-27T15:42:13.835+01:00"}
---

## Difference between single quotes and double quotes in bash
```bash
echo 'Successfully created new markdown file $file_name!'
echo "Successfully created new markdown file \"$file_name\"!"
```
> variable is not expanded in single quotes

## How to get the root path of the git repository?
```bash
git rev-parse --show-toplevel
```

## How to get all changed files in the current commit?
```bash
git diff --cached --name-status
```

## Usage of `sed` command in macOs
```bash
sed -i '' "s/^date:.*/$timestamp/" "$file_path"
```
> NOTE: Mac OS forces to always save backups when using the `sed` command in in-place change mode


## Iterate the output from the bash program
```bash
while read st file; do

	# Skip deleted files
	if [ "$st" == 'D' ]; then continue; fi

	# Only check the markdown files in the _post folder
	if [[ $file =~ \.md$ ]]; then
		check_markdown_timestamp $file
	fi
done < <(git diff --cached --name-status)
```
> `<(git diff --cached --name-status)` is ...

## How to get the current time?
```bash
timestamp_date=$(date +%Y-%m-%d)
timestamp_time=$(date +%H:%M:%S)
```

## How to check an string is empty?
```bash
if [ -z "$file_name_arg" ]; then
fi
```
> NOTE: there are many ways of checking whether a string is empty or not in bash, here it checks the string is empty


## Bash string substitution
```bash
# Contacant the full file name
delim_in_arg=" "
delim_in_file_name="-"
#source: https://stackoverflow.com/questions/13210880/replace-one-substring-for-another-string-in-shell-script
# replace all occurrences of delimiter in the argument with the expected one
file_name=${file_name_arg//$delim_in_arg/$delim_in_file_name}
file_name="$timestamp_date"-"$file_name.md"
```
> `//` forces to replace all ocurrences of the keywords with expected ones


## How to check whether a file exists or not?
```bash
if test -f "./$file_name"; then
fi
```

## How to assign a multi-line string to a variable?
```bash
# source: https://stackoverflow.com/a/37222377, create a string accrossing multi-line by utilizing the here-doc
md_header_template=$(cat <<-END
---
layout: post
title: ""
2023-12-27 08:50:12
description: ""
tags: [""]
categories: [""]
giscus_comments: true
toc:
  beginning: true
---
END

) #!NOTE: `END` flag must be placed on a line by itself
```

> NOTE: this is so called `heredoc` in bash, the `-` hyphen can be ommited.
> NOTE: "$md_header_template" with double quotes must be used to preserver the unreadable characters such as `\t`, `\n`
