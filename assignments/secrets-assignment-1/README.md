echo "password" | docker secret create password -

docker stack deploy -c drupal.yml drupal_guy