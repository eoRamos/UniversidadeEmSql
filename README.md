CREATE TABLE Alunos (
    id_aluno INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255),
    sobrenome VARCHAR(255),
    email VARCHAR(255)
);

CREATE TABLE Cursos (
    id_curso INT AUTO_INCREMENT PRIMARY KEY,
    nome_curso VARCHAR(255),
    area VARCHAR(255)
);

CREATE TABLE Matriculas (
    id_matricula INT AUTO_INCREMENT PRIMARY KEY,
    id_aluno INT,
    id_curso INT,
    data_matricula TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_aluno) REFERENCES Alunos(id_aluno),
    FOREIGN KEY (id_curso) REFERENCES Cursos(id_curso)
);

DELIMITER //
CREATE FUNCTION GerarEmail(nome VARCHAR(255), sobrenome VARCHAR(255)) 
RETURNS VARCHAR(255)
BEGIN
    DECLARE email VARCHAR(255);
    SET email = CONCAT(nome, '.', sobrenome, '@dominio.com');
    RETURN email;
END//
DELIMITER ;

DELIMITER //
CREATE PROCEDURE InserirCurso(nome_curso VARCHAR(255), area VARCHAR(255))
BEGIN
    INSERT INTO Cursos (nome_curso, area) VALUES (nome_curso, area);
END//
DELIMITER ;

DELIMITER //
CREATE PROCEDURE ObterIdCurso(nome_curso VARCHAR(255), area VARCHAR(255))
BEGIN
    SELECT id_curso FROM Cursos WHERE nome_curso = nome_curso AND area = area;
END//
DELIMITER ;

DELIMITER //
CREATE PROCEDURE MatricularAluno(nome VARCHAR(255), sobrenome VARCHAR(255), email VARCHAR(255), nome_curso VARCHAR(255), area VARCHAR(255))
BEGIN
    DECLARE aluno_existente INT;
    DECLARE curso_existente INT;
    
    SELECT COUNT(*) INTO aluno_existente FROM Matriculas M
    INNER JOIN Alunos A ON M.id_aluno = A.id_aluno
    WHERE A.nome = nome AND A.sobrenome = sobrenome;
    
    IF aluno_existente > 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Aluno já matriculado';
    ELSE
        SELECT COUNT(*) INTO curso_existente FROM Cursos WHERE nome_curso = nome_curso AND area = area;
        
        IF curso_existente > 0 THEN
            CALL ObterIdCurso(nome_curso, area);
            
            INSERT INTO Alunos (nome, sobrenome, email) VALUES (nome, sobrenome, email);
            SET @id_aluno = LAST_INSERT_ID();
            
            CALL ObterIdCurso(nome_curso, area);
            SET @id_curso = id_curso;
            
            INSERT INTO Matriculas (id_aluno, id_curso) VALUES (@id_aluno, @id_curso);
        ELSE
            SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Curso não encontrado';
        END IF;
    END IF;
END//
DELIMITER ;

CREATE FUNCTION GerarEmail(nome VARCHAR(255), sobrenome VARCHAR(255)) 
RETURNS VARCHAR(255)
BEGIN
    DECLARE email VARCHAR(255);
    SET email = CONCAT(nome, '.', sobrenome, '@dominio.com');
    RETURN email;
END;

SET @count_alunos := 1;
REPEAT
    INSERT INTO Alunos (nome, sobrenome, email)
    VALUES ('Aluno' + CAST(@count_alunos AS CHAR), 'Sobrenome' + CAST(@count_alunos AS CHAR), GerarEmail('Aluno' + CAST(@count_alunos AS CHAR), 'Sobrenome' + CAST(@count_alunos AS CHAR)));
    SET @count_alunos := @count_alunos + 1;
UNTIL @count_alunos > 200 END REPEAT;

INSERT INTO Cursos (nome_curso, area)
VALUES 
    ('Engenharia Civil', 'Engenharia'),
    ('Matemática Aplicada', 'Matemática'),
    ('Ciência da Computação', 'Tecnologia'),
    ('Biologia Molecular', 'Ciências Biológicas'),
    ('Direito Constitucional', 'Direito'),
    ('Administração de Empresas', 'Administração'),
    ('Arquitetura Sustentável', 'Arquitetura'),
    ('Medicina Veterinária', 'Medicina'),
    ('Design Gráfico', 'Design'),
    ('Psicologia Clínica', 'Psicologia'),
    ('Contabilidade Financeira', 'Contabilidade'),
    ('Música Popular', 'Música'),
    ('Física Quântica', 'Física'),
    ('Economia Internacional', 'Economia'),
    ('Língua e Literatura Inglesa', 'Linguística'),
    ('Filosofia Política', 'Filosofia'),
    ('História da Arte', 'História'),
    ('Engenharia de Software', 'Engenharia de Software'),
    ('Nutrição Clínica', 'Nutrição'),
    ('Geografia Humana', 'Geografia'),
    ('Teatro Musical', 'Teatro'),
    ('Química Orgânica', 'Química'),
    ('Educação Infantil', 'Educação'),
    ('Gestão de Projetos', 'Gestão'),
    ('Marketing Digital', 'Marketing');
