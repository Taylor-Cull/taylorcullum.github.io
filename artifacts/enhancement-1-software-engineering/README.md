""
Enhanced CRUD module for the Austin Animal Center MongoDB database.

This module provides reusable Create and Read operations for the
AnimalShelter collection.
"""

from os import getenv
from pymongo import MongoClient
from pymongo.errors import PyMongoError


class AnimalShelter:
    """Provide database access operations for the AAC animal collection."""

    def __init__(
        self,
        username=None,
        password=None,
        host=None,
        port=None,
        database="aac",
        collection="animals",
    ):
        """Initialize the MongoDB connection.

        Credentials can be supplied as arguments or through environment
        variables so they do not have to be hard-coded in the source.
        """
        self.username = username or getenv("AAC_MONGO_USER")
        self.password = password or getenv("AAC_MONGO_PASSWORD")
        self.host = host or getenv("AAC_MONGO_HOST", "localhost")
        self.port = int(port or getenv("AAC_MONGO_PORT", "27017"))
        self.database_name = database
        self.collection_name = collection

        if not self.username or not self.password:
            raise ValueError(
                "MongoDB credentials are required. Provide them as arguments "
                "or set AAC_MONGO_USER and AAC_MONGO_PASSWORD."
            )

        connection_string = (
            f"mongodb://{self.username}:{self.password}@"
            f"{self.host}:{self.port}"
        )

        try:
            self.client = MongoClient(connection_string)
            self.database = self.client[self.database_name]
            self.collection = self.database[self.collection_name]
        except PyMongoError as error:
            raise ConnectionError(
                f"Unable to connect to MongoDB: {error}"
            ) from error

    def create(self, data):
        """Insert one animal document into the collection.

        Returns True when the document is inserted successfully and
        False when the database operation fails.
        """
        if not isinstance(data, dict) or not data:
            raise ValueError("Data must be a non-empty dictionary.")

        try:
            result = self.collection.insert_one(data)
            return result.acknowledged
        except PyMongoError as error:
            print(f"Create operation failed: {error}")
            return False

    def read(self, query):
        """Return animal documents matching the supplied query."""
        if not isinstance(query, dict):
            raise ValueError("Query must be provided as a dictionary.")

        try:
            return list(self.collection.find(query, {"_id": False}))
        except PyMongoError as error:
            print(f"Read operation failed: {error}")
            return []

