    DELIMITER $$
    
    CREATE PROCEDURE insert_lines_from_blob(IN input_blob TEXT)
    BEGIN
      DECLARE line TEXT;
      DECLARE remaining TEXT;
      DECLARE newline_pos INT;
    
      SET remaining = input_blob;
    
      WHILE CHAR_LENGTH(remaining) > 0 DO
        SET newline_pos = LOCATE('\n', remaining);
    
        IF newline_pos > 0 THEN
          SET line = LEFT(remaining, newline_pos - 1);
          SET remaining = SUBSTRING(remaining, newline_pos + 1);
        ELSE
          SET line = remaining;
          SET remaining = '';
        END IF;
    
        -- Trim carriage returns if present
        SET line = REPLACE(line, '\r', '');
    
        -- Insert into table
        IF CHAR_LENGTH(line) > 0 THEN
          INSERT INTO lines_table (line_of_text) VALUES (line);
        END IF;
      END WHILE;
    END$$
    
    DELIMITER ;
