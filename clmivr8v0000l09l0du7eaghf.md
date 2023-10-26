---
title: "Merging and updating strategies in PostgreSQL"
seoTitle: "Merge & Update PostgreSQL Strategies"
seoDescription: "Optimize PostgreSQL updates with CASE..WHEN, MERGE; prevent Golang SQL injection; test using sql-mock for data syncing efficiency"
datePublished: Thu Sep 14 2023 07:59:47 GMT+0000 (Coordinated Universal Time)
cuid: clmivr8v0000l09l0du7eaghf
slug: merging-and-updating-strategies-in-postgresql
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/fPkvU7RDmCo/upload/88f0dc20fdc5100246b889c730d8ba54.jpeg
tags: postgresql, go, sql, mocking, batch-processing

---

## Introduction

At Dwarves, a few of our projects require syncing data across multiple systems. Some data gets duplicated, but the bigger problem comes with batch updates and how to handle data that differs by `id` in the same model. We'll be diving into a few ways to handle batch updates in SQL, specifically focusing on the `CASE..WHEN` method and the new `MERGE` statement introduced in PostgreSQL 15.

## Batch Updates in SQL

When dealing with large amounts of data, it's crucial to minimize the number of SQL calls to the database. One way to achieve this is by using batch updates. Batch updates allow you to **update multiple rows in a single SQL statement**, reducing the number of calls and improving performance.

### **Updating with** `CASE..WHEN`

One way to batch update is to use `CASE..WHEN`. The `CASE..WHEN` statement is a powerful tool for performing batch updates in SQL. It allows you to conditionally update multiple rows based on specific criteria. Here's a simple example of how you can use `CASE..WHEN` to update game character information:

```sql
UPDATE game_characters
SET 
  level = CASE
    WHEN character_id = 1 THEN 10
    WHEN character_id = 2 THEN 15
    ELSE level
  END,
  star = CASE
    WHEN character_id = 1 THEN 3
    WHEN character_id = 2 THEN 4
    ELSE star
  END,
  -- Add more CASE statements for other attributes here
  accuracy = CASE
    WHEN character_id = 1 THEN 95
    WHEN character_id = 2 THEN 90
    ELSE accuracy
  END;
```

In this example, we're updating the `level` column for two characters with specific `character_id` values. The `CASE..WHEN` statement checks each row for the specified conditions and updates the `level` column accordingly.

### **The** `MERGE` **Statement in PostgreSQL 15**

Another way to batch update is to use the new `MERGE` statement in PostgreSQL.

#### Prior Art

Before this, you would have to create a temporary table to mimic the target table and update it through `UPDATE ... FROM`. If you are working with an older PostgreSQL version, you can look at [my post on how to do it on TimescaleDB](https://monotykamary.hashnode.dev/a-merge-upsert-pattern-for-timescaledb) to create a similar merge pattern.

#### Using the MERGE statement

PostgreSQL 15 introduces a new statement called `MERGE`, which allows you to perform both `INSERT` and `UPDATE` operations in a single statement. This is particularly useful when you need to update data across multiple tables, as is often the case with game character information.

```sql
WITH updated_characters AS (
  SELECT character_id, level, star, -- Add more attributes here
  FROM (VALUES 
    (1, 10, 3), -- Add more attribute values for character_id 1 here
    (2, 15, 4)  -- Add more attribute values for character_id 2 here
  ) AS t(character_id, level, star) -- Add more attribute names here
)
MERGE INTO characters c
USING updated_characters uc
ON (c.character_id = uc.character_id)
WHEN MATCHED THEN
  UPDATE SET 
    level = uc.level,
    star = uc.star
    -- Add more attribute updates here
WHEN NOT MATCHED THEN
  INSERT (character_id, level, star) VALUES (uc.character_id, uc.level, uc.star); -- Add more attribute names here
```

In this example, we first create a CTE `updated_characters` containing the updated data. Then, we use the `MERGE` statement to update the `characters` table accordingly. The `MERGE` statement checks for matching rows based on the specified conditions and performs either an `UPDATE` or an `INSERT` operation.

## Doing this programmatically in Golang

Now that we have a general idea of how to do batch updates, we want to do this programmatically for one of our models. Our example will be simple and won't involve any relations. Our example will be similar to above, where we will use a model to represent a game character in our app:

```go
package model

import (
    "gorm.io/gorm"
	_ "gorm.io/gorm/dialects/postgres"
)

type GameCharacter struct {
	CharacterID   int     `gorm:"column:character_id;primary_key"`
	Level         int     `gorm:"column:level"`
	Star          int     `gorm:"column:star"`
	Growth        float64 `gorm:"column:growth"`
	HP            int     `gorm:"column:hp"`
	MP            int     `gorm:"column:mp"`
	Power         int     `gorm:"column:power"`
	Strength      int     `gorm:"column:strength"`
	Intelligence  int     `gorm:"column:intelligence"`
	Dexterity     int     `gorm:"column:dexterity"`
	Vitality      int     `gorm:"column:vitality"`
	Speed         int     `gorm:"column:speed"`
	MagicAtk      int     `gorm:"column:magic_atk"`
	PhysicalAtk   int     `gorm:"column:physical_atk"`
	PhysicalDef   int     `gorm:"column:physical_def"`
	CritRate      float64 `gorm:"column:crit_rate"`
	CritDmg       float64 `gorm:"column:crit_dmg"`
	Accuracy      float64 `gorm:"column:accuracy"`
}

func (GameCharacter) TableName() string {
	return "game_characters"
}
```

Since we're going to be manipulating strings, we're going to run into **SQL injection**. To prevent this, we're going to make sure that all the input values are sanitized. Luckily, the only inputs that could be affected are the **column names** of our `GameCharacter` struct. We can take advantage of string fields inside Go and extract it from our `gorm -> column` field:

```go
func GetValidColumns() map[string]bool {
	validColumns := make(map[string]bool)
	gameCharacter := GameCharacter{}
	gameCharacterType := reflect.TypeOf(gameCharacter)

	for i := 0; i < gameCharacterType.NumField(); i++ {
		field := gameCharacterType.Field(i)
		column := field.Tag.Get("gorm")
		columnName := strings.Split(column, ";")[0][7:]
		validColumns[columnName] = true
	}

	return validColumns
}
```

Now that we have everything, let's create the `UpdateCharacterAttributes` function using `CASE..WHEN`. We'll generalize the function by using `map` and having our key reference our character id with the value being an `interface{}` that takes in any of our valid columns:

```go
func UpdateCharacterAttributes(db *gorm.DB, updates map[int]map[string]interface{}) error {
	validColumns := GetValidColumns()
	var cases []string
	args := []interface{}{}

	for column := range updates[0] {
		if !validColumns[column] {
			return fmt.Errorf("Invalid column name: %s", column)
		}

		quotedColumn := db.Statement.Quote(column)
        caseStatement := fmt.Sprintf("%s = CASE", quotedColumn)
		for characterID, attributes := range updates {
			caseStatement += fmt.Sprintf(" WHEN character_id = ? THEN ?")
			args = append(args, characterID, attributes[column])
		}
		caseStatement += fmt.Sprintf(" ELSE %s END", quotedColumn)
		cases = append(cases, caseStatement)
	}
	setStatement := strings.Join(cases, ", ")

	sql := fmt.Sprintf("UPDATE game_characters SET %s", setStatement)

	return db.Exec(sql, args...).Error
}
```

Likewise, we will do the same for our `MergeCharacterAttributes` function.

```go
func MergeCharacterAttributes(db *gorm.DB, updates map[int]map[string]interface{}) error {
	validColumns := GetValidColumns()
	var columns, values, updateSet, insertColumns, insertValues []string
	args := []interface{}{}

	for column := range updates[0] {
		if !validColumns[column] {
			return fmt.Errorf("Invalid column name: %s", column)
		}

		quotedColumn := db.Statement.Quote(column)
        columns = append(columns, quotedColumn)
		for characterID, attributes := range updates {
			values = append(values, fmt.Sprintf("?", attributes[column]))
			args = append(args, characterID, attributes[column])
		}
		updateSet = append(updateSet, fmt.Sprintf("%s = uc.%s", quotedColumn, quotedColumn))
		insertColumns = append(insertColumns, quotedColumn)
		insertValues = append(insertValues, "?")
	}

	columnsStr := strings.Join(columns, ", ")
	valuesStr := strings.Join(values, ", ")
	updateSetStr := strings.Join(updateSet, ", ")
	insertColumnsStr := strings.Join(insertColumns, ", ")
	insertValuesStr := strings.Join(insertValues, ", ")

	sql := fmt.Sprintf(`
		WITH updated_characters AS (
			SELECT ? AS character_id, %s
			FROM (VALUES (%s)) AS t(character_id, %s)
		)
		MERGE INTO game_characters c
		USING updated_characters uc
		ON (c.character_id = uc.character_id)
		WHEN MATCHED THEN
			UPDATE SET %s
		WHEN NOT MATCHED THEN
			INSERT (%s) VALUES (?, %s);
	`, columnsStr, valuesStr, columnsStr, updateSetStr, insertColumnsStr, insertValuesStr)

	return db.Exec(sql, args...).Error
}
```

### Testing it all together with `sql-mock`

We'll now create some unit tests to test our new SQL functions. `sql-mock` is very useful here to allow us to test the function to an expected executed SQL statement. Let's create our test for our `UpdateCharacterAttributes` function.

```go
func (suite *GameCharacterTestSuite) TestUpdateCharacterAttributes() {
	updates := map[int]map[string]interface{}{
		1: {"level": 10, "star": 3},
		2: {"level": 15, "star": 4},
	}

	suite.mock.ExpectBegin()
	suite.mock.ExpectExec("UPDATE game_characters SET level = CASE WHEN character_id = \\? THEN \\? ELSE level END, star = CASE WHEN character_id = \\? THEN \\? ELSE star END").
		WithArgs(1, 10, 2, 15, 1, 3, 2, 4).
		WillReturnResult(sqlmock.NewResult(0, 2))
	suite.mock.ExpectCommit()

	err := UpdateCharacterAttributes(suite.DB, updates)
	assert.NoError(suite.T(), err)
}
```

We now have an idea of how the function's inputs for updates will look like. Just like we stated above, we use the id of the character as our **key** and an object containing our valid columns and values we want to update with as our **value**.

Let's do the same for our `MergeCharacterAttributes` function:

```go
func (suite *GameCharacterTestSuite) TestMergeCharacterAttributes() {
	updates := map[int]map[string]interface{}{
		1: {"level": 10, "star": 3},
		2: {"level": 15, "star": 4},
	}

	suite.mock.ExpectBegin()
	suite.mock.ExpectExec("WITH updated_characters AS \\(SELECT \\? AS character_id, level, star FROM \\(VALUES \\(\\?, \\?, \\?\\), \\(\\?, \\?, \\?\\)\\) AS t\\(character_id, level, star\\)\\) MERGE INTO game_characters c USING updated_characters uc ON \\(c.character_id = uc.character_id\\) WHEN MATCHED THEN UPDATE SET level = uc.level, star = uc.star WHEN NOT MATCHED THEN INSERT \\(character_id, level, star\\) VALUES \\(\\?, \\?, \\?\\);").
		WithArgs(1, 10, 3, 2, 15, 4).
		WillReturnResult(sqlmock.NewResult(0, 2))
	suite.mock.ExpectCommit()

	err := MergeCharacterAttributes(suite.DB, updates)
	assert.NoError(suite.T(), err)
}
```

## Conclusion

With that, we explored some different ways to perform batch updates in SQL, focusing on `CASE..WHEN` and the new `MERGE` statement in PostgreSQL 15. The columns are whitelisted for input sanitation and scoped to avoid SQL injections. In addition, mocking can help you unit-test your queries with peace of mind (however, YMMV when using `sql-mock`). Hopefully, this post inspired you to try out a different way to compose your queries and maybe create your own implementation for your use case.

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    
* *Visit our* [*GitHub*](https://github.com/dwarvesf)