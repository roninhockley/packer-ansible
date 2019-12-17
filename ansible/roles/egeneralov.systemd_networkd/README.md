# egeneralov.systemd-networkd

Move from networking to systemd-networkd (bridge will be created)

## Requirements

Debian 9.

## Example Playbook

Run only localy.

    ---
    
    - hosts: localhost
      remote_user: root
      roles:
        - egeneralov.systemd-networkd

## License

MIT

## Author Information

Eduard Generalov <eduard@generalov.net>
