        DELIMITER $$
        
        CREATE PROCEDURE insert_single_folder_path(IN full_path VARCHAR(1000))
        BEGIN
          DECLARE idx INT DEFAULT 1;
          DECLARE part VARCHAR(255);
          DECLARE parent INT DEFAULT NULL;
          DECLARE current_id INT;
        
          DROP TEMPORARY TABLE IF EXISTS temp_parts;
          CREATE TEMPORARY TABLE temp_parts (
            part_order INT,
            part_name VARCHAR(255)
          );
        
          WHILE LOCATE('/', full_path) > 0 DO
            SET part = SUBSTRING_INDEX(full_path, '/', 1);
            INSERT INTO temp_parts (part_order, part_name)
            VALUES ((SELECT IFNULL(MAX(part_order), 0) + 1 FROM temp_parts), part);
            SET full_path = SUBSTRING(full_path, LOCATE('/', full_path) + 1);
          END WHILE;
        
          -- Insert the last part (leaf)
          IF full_path != '' THEN
            INSERT INTO temp_parts (part_order, part_name)
            VALUES ((SELECT IFNULL(MAX(part_order), 0) + 1 FROM temp_parts), full_path);
          END IF;
        
          -- Now insert folders into `folders` table
          DECLARE done INT DEFAULT FALSE;
          DECLARE cur CURSOR FOR SELECT part_name FROM temp_parts ORDER BY part_order;
          DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
        
          OPEN cur;
          read_loop: LOOP
            FETCH cur INTO part;
            IF done THEN
              LEAVE read_loop;
            END IF;
        
            -- Look for existing folder under current parent
            SELECT id INTO current_id
            FROM folders
            WHERE folder_name = part AND (parent_id <=> parent)
            LIMIT 1;
        
            IF current_id IS NULL THEN
              INSERT INTO folders (folder_name, parent_id)
              VALUES (part, parent);
              SET current_id = LAST_INSERT_ID();
            END IF;
        
            SET parent = current_id;
          END LOOP;
          CLOSE cur;
        END$$
        
        DELIMITER ;
