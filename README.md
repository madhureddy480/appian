    DELIMITER $$
    
    CREATE PROCEDURE insert_single_folder_path(IN full_path VARCHAR(1000))
    BEGIN
      DECLARE part VARCHAR(255);
      DECLARE parent INT DEFAULT NULL;
      DECLARE current_id INT;
      DECLARE done INT DEFAULT FALSE;
      DECLARE part_order_counter INT DEFAULT 1;
    
      -- Cursor declarations must come before any SQL
      DECLARE cur CURSOR FOR SELECT part_name FROM temp_parts ORDER BY part_order;
      DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
      -- Temp table for holding split path parts
      DROP TEMPORARY TABLE IF EXISTS temp_parts;
      CREATE TEMPORARY TABLE temp_parts (
        part_order INT,
        part_name VARCHAR(255)
      );
    
      -- Split the path by '/' and populate temp_parts
      WHILE LOCATE('/', full_path) > 0 DO
        SET part = SUBSTRING_INDEX(full_path, '/', 1);
        INSERT INTO temp_parts (part_order, part_name) VALUES (part_order_counter, part);
        SET part_order_counter = part_order_counter + 1;
        SET full_path = SUBSTRING(full_path, LOCATE('/', full_path) + 1);
      END WHILE;
    
      IF full_path != '' THEN
        INSERT INTO temp_parts (part_order, part_name) VALUES (part_order_counter, full_path);
      END IF;
    
      -- Insert into folders table step by step
      OPEN cur;
      read_loop: LOOP
        FETCH cur INTO part;
        IF done THEN
          LEAVE read_loop;
        END IF;
    
        -- Check if folder exists under the current parent
        SELECT id INTO current_id
        FROM folders
        WHERE folder_name = part AND (parent_id <=> parent)
        LIMIT 1;
    
        -- If not found, insert it
        IF current_id IS NULL THEN
          INSERT INTO folders (folder_name, parent_id)
          VALUES (part, parent);
          SET current_id = LAST_INSERT_ID();
        END IF;
    
        SET parent = current_id;  -- Prepare parent for next loop
      END LOOP;
      CLOSE cur;
    END$$
    
    DELIMITER ;
      
