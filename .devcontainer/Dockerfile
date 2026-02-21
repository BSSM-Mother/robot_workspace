FROM althack/ros2:jazzy-dev

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
   && apt-get -y install --no-install-recommends \
      python3-ament-cppcheck \
      python3-ament-cpplint \
      python3-ament-flake8 \
      python3-ament-pep257 \
      python3-ament-cmake-lint \
      python3-ament-xmllint \
   && apt-get autoremove -y \
   && apt-get clean -y \
   && rm -rf /var/lib/apt/lists/*
ENV DEBIAN_FRONTEND=dialog

# Set up auto-source of workspace for ros user
ARG WORKSPACE
RUN echo "if [ -f ${WORKSPACE}/install/setup.bash ]; then source ${WORKSPACE}/install/setup.bash; fi" >> /home/ros/.bashrc