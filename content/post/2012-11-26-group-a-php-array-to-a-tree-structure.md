---
author: admin
comments: true
date: 2012-11-26 20:40:38+00:00
layout: post
slug: group-a-php-array-to-a-tree-structure
title: Group a php array to a tree structure
wordpress_id: 286
categories:
- Software
tags:
- php
---

Last week I needed a php method which helps me with converting a Mysql result into a grouped tree structure. The grouping was supposed to be on certain column names from my database result. A quick Google search didn't give me any useful results so sharing this seems a good option.

Disclaimer: I'm not a professional php'er so there's a big chance that this is not the most easy or fastest way. But anyway, it works well for big result sets.

``` php
$original = array
(
    "1" => array
    (
        "id" => 1,
        "type" => "mtb",
        "brand" => "scott",
        "name" => "scott1"
    ),
    array
    (
        "id" => 2,
        "type" => "mtb",
        "brand" => "cube",
        "name" => "cube1"
    ),
    array
    (
        "id" => 3,
        "type" => "race",
        "brand" => "cube",
        "name" => "cuberace1"
    ),
    array
    (
        "id" => 4,
        "type" => "race",
        "brand" => "giant",
        "name" => "giant1"
    ),
    array
    (
        "id" => 5,
        "type" => "mtb",
        "brand" => "cube",
        "name" => "cube2"
    )
);

echo "<pre>";
print_r($original);
echo "</pre>";

$converted = array();
$keys = array('type', 'brand');
$level = 0;

$converted = createTree($original, $keys);

echo "<pre>";
print_r($converted);
echo "</pre>";

function createTree($original, $keys, $level = 0)
{
  $converted = array();
  $key = $keys[$level];
  $isDeepest = sizeof($keys) -1 == $level;

  $level++;
  $filtered = array();

  foreach ($original as $k => $subArray)
  {
    $thisLevel = $subArray[$key];
    if($isDeepest) $converted[$thisLevel][] = $subArray;
    else $converted[$thisLevel] = array();

    $filtered[$thisLevel][] = $subArray;
  }

  if(!$isDeepest)
  {
    foreach (array_keys($converted) as $value)
    {
      $converted[$value] = createTree($filtered[$value], $keys, $level);
    }
  }

  return $converted;
}
```



The output:
``` html
Array
(
    [mtb] => Array
        (
            [scott] => Array
                (
                    [0] => Array
                        (
                            [id] => 1
                            [type] => mtb
                            [brand] => scott
                            [name] => scott1
                        )

                )

            [cube] => Array
                (
                    [0] => Array
                        (
                            [id] => 2
                            [type] => mtb
                            [brand] => cube
                            [name] => cube1
                        )

                    [1] => Array
                        (
                            [id] => 5
                            [type] => mtb
                            [brand] => cube
                            [name] => cube2
                        )

                )

        )

    [race] => Array
        (
            [cube] => Array
                (
                    [0] => Array
                        (
                            [id] => 3
                            [type] => race
                            [brand] => cube
                            [name] => cuberace1
                        )

                )

            [giant] => Array
                (
                    [0] => Array
                        (
                            [id] => 4
                            [type] => race
                            [brand] => giant
                            [name] => giant1
                        )

                )

        )

)
```
