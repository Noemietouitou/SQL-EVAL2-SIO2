Fiche Reponses:




1. ## Trigger de Mise à Jour de la disponibilité des Voitures lors d'une Réservation

   - Nom du Trigger : tr_update_disponibilite_voiture
   - Événement : AFTER INSERT sur la table Réservations
   - Objectif : Marque une voiture en indisponibilité lors de sa reservation 
   
   - Code SQL :
DELIMITER //

CREATE TRIGGER tr_update_disponibilite_voiture
AFTER INSERT ON Réservations
FOR EACH ROW

BEGIN
UPDATE Voitures
SET disponible = 0
WHERE id = NEW.voiture_id 
END //
DELIMITER ;





2. ## Trigger de Vérification de l'Âge du Client avant une Réservation

   - Nom du Trigger : tr_verification_age_before_reservation
   - Événement : BEFORE INSERT sur la table Réservations
   - Objectif : Verifie que le client a bien au minimum 21 ans et affiche un message d'erreur sinon

   - Code SQL :

   DELIMITER //

   CREATE TRIGGER tr_verification_age_before_reservation
   BEFORE INSERT ON Réservations
   FOR EACH ROW

   BEGIN
   DECLARE age INT;
   SELECT Clients.age INTO age
   FROM Clients
   WHERE id = NEW.client_id;

   IF age < 21 THEN
   SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Age minimum pour louer une voiture est 21 ans.';
   END IF;
   END //
   DELIMITER ;


3. ## Trigger de Vérification de la Validité du Permis de Conduire avant la création d'un nouveau client
   - Nom du Trigger : tr_verification_permis
   - Événement : BEFORE INSERT sur la table Clients
   - Objectif : Verifie l'existance, l'unicite et la validite du permis de conduire avant insertion du Client

   - Code SQL :

    DELIMITER //

    CREATE TRIGGER tr_verification_permis
    BEFORE INSERT ON Clients
    FOR EACH ROW
    BEGIN

    IF EXISTS (SELECT 1 FROM Clients WHERE permis_conduire = NEW.permis_conduire) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Le permis de conduire doit être unique.';
    END IF;
    IF NEW.permis_conduire IS NULL OR NEW.permis_conduire = '' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Le client doit avoir un permis de conduire.';
    END IF;
    IF LENGTH(NEW.permis_conduire) != 15 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Le numéro de permis de conduire doit avoir 15 caractères.';
    END IF;
    IF NOT NEW.permis_conduire REGEXP '^[A-Z0-9]{15}$' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Le permis de conduire doit être alphanumérique et comporter 15 caractères.';
    END IF;

    END //

    DELIMITER ;



4. ## Trigger de Vérification de la Disponibilité de la Voiture avant une Réservation

   - Nom du Trigger : tr_verification_disponibilite_voiture
   - Événement : BEFORE INSERT sur la table Réservations
   - Objectif : S'assure de la disponibilite de la voiture et qu'elle n'est pas en maintenance

   - Code SQL :

    DELIMITER //

    CREATE TRIGGER tr_verification_disponibilite_voiture
    BEFORE INSERT ON Réservations
    FOR EACH ROW
    BEGIN

    DECLARE dispo INT;
    DECLARE maintenance INT;

    SELECT Voitures.disponible, Voitures.en_maintenance
    INTO dispo, maintenance
    FROM Voitures
    WHERE Voitures.id = NEW.voiture_id;

    IF dispo = 0 OR maintenance = 1 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La voiture n\'est pas disponible à la réservation';
    END IF;

    END //
    DELIMITER ;


5. ## Trigger pour Éviter les Chevauchements de Réservations sur la Même Voiture

    - Nom du Trigger : tr_non_chevauchement
    - Événement : BEFORE INSERT sur la table Réservations
    - Objectif : Verifie que la voiture n'est pas deja reservee sur la periode et envoie un message sinon 

    - Code SQL :

    DELIMITER //

    CREATE TRIGGER tr_verification_chevauchement
    BEFORE INSERT ON Réservations
    FOR EACH ROW
    BEGIN

    DECLARE reservation_chevauchee INT;
    SELECT COUNT(*)
    INTO reservation_chevauchee
    FROM Réservations
    WHERE voiture_id = NEW.voiture_id
    AND (
        (NEW.date_debut BETWEEN date_debut AND date_fin) OR (date_debut BETWEEN NEW.date_debut AND NEW.date_fin) OR (date_fin BETWEEN NEW.date_debut AND NEW.date_fin) 
    );

    IF reservation_chevauchee > 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'La voiture est déjà réservée pendant cette période.';
    END IF;

    END //

    DELIMITER ;


